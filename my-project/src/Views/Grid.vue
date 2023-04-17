<template>
<div>
    <!-- API Data  -->
 <!--    <ul>
      <li v-for="result in results" :key="result.ksamsok_id">
        {{ result.coordinates.coordinates }}
      </li>
    </ul>  -->

<div class="top">  
  <h1 class="title">Swedish <br><div class="emph">Rock Art </div><br>Research<br> Archive</h1>
</div>
<!--  <div class="w-1/2 m-8 mb-32 flex">
  <div class="relative w-full">
    <input type="text" id="search" name="search" placeholder="Search Image Database" class="border border-gray-400 py-3 pl-6 pr-10 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-400 w-full text-black drop-shadow-lg">
    <svg xmlns="http://www.w3.org/2000/svg" class="dropdown-svg h-5 w-5 absolute cursor-pointer top-4 right-4 text-gray-400" fill="currentColor" viewBox="0 0 24 24" @click="showDropDown" stroke="currentColor">
      <path fill-rule="evenodd" d="M13.853 12.146a1 1 0 0 1-1.414 1.414l-1.147-1.147a4.5 4.5 0 1 1 1.414-1.414l1.147 1.147zm3.708 3.708a1 1 0 0 1-1.414 1.414l-3.8-3.8a6 6 0 1 1 1.414-1.414l3.8 3.8zM6.5 10a3.5 3.5 0 1 1 7 0 3.5 3.5 0 0 1-7 0z" clip-rule="evenodd"></path>
    </svg>
  </div>
  <div id="dropdown" class="absolute z-10 top-12 right-[200px] py-2 w-[400px] bg-white rounded-md shadow-xl" v-if="showDropdown">
    <a href="#" class="block px-4 py-2 text-sm text-gray-700 hover:bg-blue-500 hover:text-white">Option 1</a>
    <a href="#" class="block px-4 py-2 text-sm text-gray-700 hover:bg-blue-500 hover:text-white">Option 2</a>
    <a href="#" class="block px-4 py-2 text-sm text-gray-700 hover:bg-blue-500 hover:text-white">Option 3</a>
    <a href="#" class="block px-4 py-2 text-sm text-gray-700 hover:bg-blue-500 hover:text-white">Option 4</a>
    <a href="#" class="block px-4 py-2 text-sm text-gray-700 hover:bg-blue-500 hover:text-white">Option 5</a>
    <a href="#" class="block px-4 py-2 text-sm text-gray-700 hover:bg-blue-500 hover:text-white">Option 6</a>
    <a href="#" class="block px-4 py-2 text-sm text-gray-700 hover:bg-blue-500 hover:text-white">Option 7</a>
    <a href="#" class="block px-4 py-2 text-sm text-gray-700 hover:bg-blue-500 hover:text-white">Option 8</a>
  </div>
</div> -->

  <!-- Start of Container -->
  <div class="split-container main-color">
  <div class="flex" style="height:calc(100vh - 230px)">
  <!-- Panel 1 -->
  <div  id="split-0" class="flex-grow flex flex-col justify-between"
       :class="{ 'w-1/3': showThreePanels, 'w-1/2': !showThreePanels }">
       
       <div id="search-interface" class="">
       <h2 class="input-unpad mb-0">
      <input type="search" id="search" name="search" placeholder="Search Archive..." class="">
       </h2>
      </div>
      <div id="filter-interface"> 
        <div class="filter-text"> Filter suggestions: </div><div class="tag-example">boat</div><div class="tag-example">animal</div> <div class="tag-example">warrior</div> <div class="tag-example">weapon</div>
      </div>

      <Map :coordinates="results" @raa-id-selected="selectedRaaId = $event"></Map>
  
      <div style="display:flex;  align-items: center; justify-content: center;">
      <div class="ui-map-info ui-overlay">
        Filter the archive by adjusting the view or select a site
      </div>
    </div>
  </div>
  <!-- Panel 2 -->
  <div id="split-1" class="flex-grow overflow-auto main-color" 
       :class="{ 'w-1/3': showThreePanels, 'w-1/2': !showThreePanels }"
       @click="toggleThreePanels">
 <!--  <div class="relative">
  <button class="absolute top-0 right-0 m-2 p-2 bg-blue-500 text-white rounded-lg">
  </button> -->

   

  <div class="">
  <div class="">
  
    <MasonryGrid :raa-id="selectedRaaId"></MasonryGrid>

    <div style="display:flex;  align-items: center; justify-content: center;">
    <div class="ui-mode ui-overlay">
        <div class="item selected">Gallery</div>
        <div class="item">Data</div>
      </div>
      <div class="ui-numbers ui-overlay">
        xxx objects
      </div>
    </div>
  </div>
 
</div>
</div>

<!-- Panel 3 -->
<transition name="slide">
<div id="split-2" class="flex-grow main-color"
      :class="{ 'w-1/3': showThreePanels, 'w-0': !showThreePanels }" v-show="showThreePanels">
    <button @click="toggleThreePanels" class="btn btn-circle m-2">
      <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
    </button>
</div>
</transition>

<div class="split-container-top"></div>
</div>
</div>
<!-- End of Container -->

</div>
</template>

<script lang="ts">
import Map from '../components/Map.vue'
import { defineComponent } from 'vue'
import Split from 'split.js';
import MasonryGrid from '../components/MasonryGrid.vue'

export default defineComponent({
  components: {
    Map, MasonryGrid
  },
  data() {
    return {
      items: [],
      results: [],
      showDropdown: false,
      showThreePanels: false,
      selectedRaaId: null,
    }
  },
  mounted() {
  
    Split(['#split-0', '#split-1', '#split-2'], {
    minSize: [500, 300],
    dragInterval: 1,
    gutterSize: 10,
    gutterAlign: 'start',
  }),
   /*    fetch('https://diana.dh.gu.se/api/shfa/site/?format=json&limit=25')
      .then(response => response.json())
      .then(data => {
        // Set the results data to the retrieved JSON data
        this.results = data.results;
      });  */

    document.addEventListener('click', this.closeDropdown);
  },
  methods: {
    toggleThreePanels() 
    {
      this.showThreePanels = !this.showThreePanels;
    },
    showDropDown()
    {
      this.showDropdown = !this.showDropdown;
    },
  },
  created() {
      async function fetchAllData(url, limit = 500, offset = 0, results = []) {
      const response = await fetch(`${url}?format=json&limit=${limit}&offset=${offset}`);
      const data = await response.json();

      const updatedResults = results.concat(data.results);

      if (data.next) {
        return fetchAllData(url, limit, offset + limit, updatedResults);
      } else {
        return updatedResults;
      }
    }

    // Usage
      fetchAllData('https://diana.dh.gu.se/api/shfa/site/')
      .then(data => {
        console.log('Fetched all data:', data);
        this.results = data; // Set the results data property
      })
      .catch(error => {
        console.error('Error fetching data:', error);
      });

      },
})
</script>


<style scoped>
.title{
  line-height:0.80;
  font-size:52px;
  font-weight:600;
  padding:25px 40px;
}

.title .emph{
  display:inline;
  color:rgb(180, 0, 0);
}

.top{
  height:230px;
  z-index:1000;

}

.main-color{
  background-color:rgb(65,65,65);

}

.split-container{
  overflow:hidden !important;
  

}

.split-container-top{
  height:calc(100% - 230px) ;
  width:100%;
  position:absolute;
  box-shadow: inset 0rem 2rem 2rem rgba(0, 0, 0, 0.4)!important;
  pointer-events:none;
}

#split-0{
padding:30px 15px 35px 40px;

}


#split-1{
padding:0px 15px 0 15px;

}



#search-interface{
  margin-bottom:10px;

 
}
#filter-interface{
  min-height:100px;
  padding:10px 0px 0px 0px;
  color:white;
  z-index:1000;
  margin-left:-5px;
}

.filter-text{
float:left;
 padding: 5px 10px;
 border-radius:5px;
 margin-left:3px;
 margin-bottom:5px;
}

.tag-example{
float:left;
 background-color: rgb(90,90,90);
 padding: 5px 10px;
 border-radius:5px;
 margin-left:10px;
 margin-bottom:5px;

 cursor:pointer;
}

.tag-example:hover{

  background-color: rgb(170,70,70);

 cursor:pointer;
}



#search {
  flex: 1;
  display: flex;
  flex-direction: column;
  margin-left: 0px;
  margin-right: 0px;
  border-radius:8px;
  margin-top:10px;
  padding:10px 15px;
  background-color:rgb(45,45,45);
 
}

input[type="search"]::-webkit-search-cancel-button {
  -webkit-appearance: none;
  height: 0.5em;
  width: 0.5em;
  border-radius: 50em;
  background: url(../../public/interface/input-cancel-x.svg) no-repeat 50% 50%;
  background-size: contain;
  opacity: 1.0;
  pointer-events: none;
}

input[type="search"]:focus::-webkit-search-cancel-button {
  opacity: 1.0;
  pointer-events: all;
  filter: invert(1);
}

h2 {
  display: flex;
  color: white;
  font-size: 30px;
  font-weight: 400;
  line-height: 0.8;
  
}

input:focus{
    outline: none;
}

h2 input {
  flex: 1;
  min-width: 3em;
  color: white;
  font-weight: 400;
  z-index: 35;
}




h2 input:hover,
h2 input:focus,
h2 input:not(:placeholder-shown) {
  background-color: black;
}


.slide-enter-active,
.slide-leave-active {
  transition: transform 200ms ease-in;
}

.slide-enter-from,
.slide-leave-to {
  transform: translateX(100%);
}

.slide-enter-to,
.slide-leave-from {
  transform: translateX(0);
}

.dropdown-svg {
  transition: transform 0.2s;
  width: 24px;
  height: 24px;
}

.dropdown-svg:hover,
.dropdown-svg:focus {
  transform: scale(1.1);
}

.ui-overlay {
z-index: 100;
position:absolute;
border-radius: 10px;
font-size: 18px;
font-weight: 500;
color: white;
background-color: rgba(0, 0, 0, 0.4);
backdrop-filter: blur(5px);
}

.ui-mode {
top: 260px;
padding: 4px 10px 4px 10px;
background-color: rgba(0, 0, 0, 0.4);
}

.ui-mode .item {
cursor: pointer;
display: inline;
padding: 0px 15px 0px 15px;
}

.ui-mode .selected{
font-weight: 500;
color: rgb(150,200,255);
}

.ui-numbers {
padding: 2px 15px 6px 15px;
text-align: center;
bottom: 30px;
margin-top: calc(100% - 100px);
}

.ui-map-info {
padding: 2px 15px 6px 15px;
text-align: center;
bottom: 50px;
margin-top: calc(100% - 100px);
}




</style>


