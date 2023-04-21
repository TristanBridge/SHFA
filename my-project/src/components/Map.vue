<template>

 <div id="map">
</div>


</template>

<script>
import Map from 'ol/Map';
import View from 'ol/View';
import TileLayer from 'ol/layer/Tile';
import OSM from 'ol/source/OSM';
import { fromLonLat } from 'ol/proj';
import VectorLayer from 'ol/layer/Vector';
import VectorSource from 'ol/source/Vector';
import Feature from 'ol/Feature';
import Point from 'ol/geom/Point';
import Icon from 'ol/style/Icon';
import Style from 'ol/style/Style';
import {toLonLat} from 'ol/proj';
import { Cluster } from 'ol/source';
import { Circle as CircleStyle, Fill, Stroke, Text } from 'ol/style';
import debounce from 'lodash/debounce';

export default {
  name: 'MapComponent',
  props: {
    coordinates: {
      type: Array,
      default: () => [],
    },
    bbox: {
      type: Array,
      default: () => [],
    },
  },
data() {
  return {
    map: null,
    vectorLayer: null,
    iconStyle: null,
    clickedRaaId: null,
    results: [],
  };
},
mounted() {
  try {
    this.initMap(); // Initialize the map on component mount
    this.$nextTick(() => {
      this.updateCoordinates(); // Update the map markers on component mount
    });
  } catch (error) {
    console.error('Error in mounted hook:', error);
  }
},
created() {
  this.fetchData();
},
watch: {
  results: {
    deep: true,
    handler() {
      this.updateCoordinates(); // Update the map markers when results change
    },
  },
  bbox: {
    deep: true,
    handler() {
      this.fetchData(); // Fetch data when the bounding box changes
    },
  },
},
  methods: {
    updateBbox: debounce(function() {
      const extent = this.map.getView().calculateExtent(this.map.getSize());
      const bottomLeft = toLonLat([extent[0], extent[1]]);
      const topRight = toLonLat([extent[2], extent[3]]);
      const newBbox = [bottomLeft[0], bottomLeft[1], topRight[0], topRight[1]];
      this.$emit('update-bbox', newBbox);
      }, 300),

    async fetchDataByBbox() {
      if (this.bbox.length === 4) {  // Construct the API URL with the given bounding box coordinates
        const [minX, minY, maxX, maxY] = this.bbox;
        let url = `https://diana.dh.gu.se/api/shfa/geojson/site/?in_bbox=${minX},${minY},${maxX},${maxY}&limit=5`;
        let allFeatures = [];

        const fetchResults = async (url) => {
          try {
            const response = await fetch(url);
            const data = await response.json();

            if (data && data.features) {
              allFeatures.push(...data.features);

            if (data.next) { //if there is a "next" URL, recursively fetch the next set of data
              let fixedNextUrl = data.next.replace('http://', 'https://');
              fixedNextUrl = decodeURIComponent(fixedNextUrl);
              await fetchResults(fixedNextUrl);
            }

            } else {
              console.error('Unexpected API response:', data);
            }
          } catch (error) {
            console.error('Error fetching data:', error);
          }
        };

        await fetchResults(url);

        // Map the retrieved features to a new array of objects with coordinates and raa_id properties
        this.results = allFeatures.map((feature) => ({
          coordinates: feature.geometry.coordinates,
          raa_id: feature.properties.raa_id,
        }));
      }
    },

async fetchData() {
  // Fetch data from the JSON file
  let jsonData = [];
  try {
    const response = await fetch('/api/reduced_data.json');
    jsonData = await response.json();
  } catch (error) {
    console.error('Error fetching reduced data:', error);
  }

  // Fetch data from the API using the current bounding box, if available
  let apiData = [];
  if (this.bbox.length === 4) {
    try {
      console.log('fetching...')
      const previousResultsLength = this.results.length;
      await this.fetchDataByBbox();
      apiData = this.results.slice(previousResultsLength); // Get the new API data from the results
    } catch (error) {
      console.error('Error fetching data using bounding box:', error);
    }
  }

  // Combine JSON data with API data
  this.results = [...jsonData, ...apiData];

  // If new data was fetched from the API, update the coordinates
  if (apiData.length > 0) {
    this.updateCoordinates();
  }
},

   initMap() {
    this.map = new Map({
    target: 'map',
    layers: [
      new TileLayer({
        source: new OSM()
      })
    ],
    view: new View({
      center: fromLonLat([11.35, 58.73]), //Default center of the map
      zoom: 13 // Default zoom level of the map
    })
  });
      // Initialize the map marker style
      this.iconStyle = new Style({
        image: new Icon({
          src: '/interface/assets/marker-gold.svg',
          scale: 1,
          anchor: [0.5, 1],
          anchorXUnits: 'fraction',
          anchorYUnits: 'fraction'
        })
      });

  // Check if coordinates are defined before creating the features
  const features = this.results.length ? this.results.map(result => {
    const coord = result.coordinates;
    return new Feature({
      geometry: new Point(fromLonLat([coord[0], coord[1]]))
    });
  }) : [];

  // Creates the new layer for the pins
  const vectorSource = new VectorSource({
    features: features
  });

  this.vectorLayer = new VectorLayer({
    source: vectorSource
  });

  this.map.addLayer(this.vectorLayer);

    // Add 'click' event listener
      this.map.on('click', (event) => {
      this.map.forEachFeatureAtPixel(event.pixel, (feature) => {
      const featuresInCluster = feature.get('features');
      if (featuresInCluster.length === 1) {
        const raa_id = featuresInCluster[0].get('raa_id');
        console.log('Clicked raa_id:', raa_id);
        this.clickedRaaId = raa_id;
        this.$emit('raa-id-selected', raa_id);
      } else {
        const coordinates = feature.getGeometry().getCoordinates();
        this.map.getView().setCenter(coordinates);
        this.map.getView().setZoom(this.map.getView().getZoom() + 1);
      }
    });
  });

  // Add 'moveend' event listener to the map to update the bounding box
  this.map.on('moveend', () => {
    this.updateBbox();
  });
},

updateCoordinates() {
  const pointSource = new VectorSource({
    features: this.results.map(result => { 
      const coordinates = result.coordinates;
      const feature = new Feature({
        geometry: new Point(fromLonLat([coordinates[0], coordinates[1]]))
      });
      feature.set('raa_id', result.raa_id); // Add raa_id as a property of the feature
      feature.setStyle(this.iconStyle); // Apply the iconStyle to the feature
      return feature;
    })
  });


  const clusterSource = new Cluster({
    distance: 40, // Adjust this value to control the clustering distance
    source: pointSource,
  });

  const clusterLayer = new VectorLayer({
    source: clusterSource,
    style: this.createClusterStyle,
  });

  // Remove the previous vectorLayer
  if (this.vectorLayer) {
    this.map.removeLayer(this.vectorLayer);
  }

  // Set the new vectorLayer to the clusterLayer and add it to the map
  this.vectorLayer = clusterLayer;
  this.map.addLayer(this.vectorLayer);
},

createClusterStyle(feature) {
  const size = feature.get('features').length;
  if (size === 1) {
    return this.iconStyle; // Return the individual pin style when cluster size is 1
  } else {
    const style = new Style({
      image: new CircleStyle({
        radius: 10 + Math.min(size, 50) * 0.1,
        fill: new Fill({ color: '#3399CC' }),
        stroke: new Stroke({
          color: '#fff',
          width: 2,
        }),
      }),
      text: new Text({
        text: size.toString(),
        fill: new Fill({ color: '#fff' }),
      }),
    });

    return style;
  }
},

  },
}
</script>

<style scoped>
#map {
  z-index: 40; /* Fixes border-radius in Safari. */
  width: 100%;
  height: 100%;
  padding:0px 0px 0px 0px;
  border-radius:10px;
  overflow:hidden!important;
  box-shadow: 0px 5px 45px rgba(0, 0, 0, 0.5)!important;
  filter:contrast(130%) grayscale(80%) brightness(0.9);
}


</style>
