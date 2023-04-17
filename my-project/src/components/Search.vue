<template>
    <div>
    <div id="search-interface" class="">
       <h2 class="input-unpad mb-0">
      <input type="search" id="search" name="search" placeholder="Search Archive..." class="">
       </h2>
      </div>
      <div id="filter-interface"> 
        <div class="filter-text"> Filter suggestions: </div><div class="tag-example">boat</div><div class="tag-example">animal</div> <div class="tag-example">warrior</div> <div class="tag-example">weapon</div>
      </div>
    </div>
</template>

<script>
export default {
  name: 'Search',
  data() {
  }
}
</script>

<style scoped>


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
  background: url(../../interface/input-cancel-x.svg) no-repeat 50% 50%;
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

</style>