<script setup>
import { ref } from 'vue';
import axios from "axios"
import { useRouter } from "vue-router"
import CityList from '../components/CityList.vue';
import CityCardSkeleton from '../components/CityCardSkeleton.vue';

const mapboxAPIKey = "pk.eyJ1IjoiaW1kc3JzIiwiYSI6ImNsazBleXpwNTA3MTUzcmw5Mmx1eGhnaG0ifQ.-akVkpcbT1okVirMPS1aCw";
const searchQuery = ref("");
const queryTimeout = ref(null);
const mapboxSearchResults = ref(null);
const searchError = ref(null);

const router = useRouter();
const previewCity = (searchResult) => {
  console.log(searchResult);
  const [city, state] = searchResult.place_name.split(",")
  console.log(city, state);
  router.push({
    name: 'cityView',
    params: {
      state: state.replaceAll(" ", ""), 
      city: city,
    },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true,
    }
  })
}

const getSearchResults = () => {
  //console.log("hello");
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    //console.log("hello2", searchQuery.value);
    if(searchQuery.value !== ""){
      //console.log("hello3", searchQuery.value);
      try{
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
          );
          mapboxSearchResults.value = result.data.features;
          console.log(mapboxSearchResults.value);
          //console.log("hello4", searchQuery.value);
          console.log(searchError.value)
        }
        catch{
          searchError.value = true;
          console.log(searchError.value);
        }
        return;
    }
      mapboxSearchResults.value = null;
      //console.log(mapboxSearchResults.value);
      return;
  }, 300);
}
</script>

<template>
  <main class="container-class text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text" placeholder="Search for a City or State"
      v-model="searchQuery"
      @input="getSearchResults"
      class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary
      focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">
      <ul class="absolute bg-weather-secondary text-white w-full
      shadow-md py-2 px-1 top-[66px]"
      v-if="mapboxSearchResults">
      <p class="py-2" v-if="searchError">
        Sorry, something went wrong!
      </p>
      <p class="py-2" v-if="!searchError && mapboxSearchResults.length === 0">
        Try a different term!
      </p>
      <template v-else>
        <li v-for="searchResult in mapboxSearchResults"
        :key="searchResult.id"
        class="py-2 cursor-pointer"
        @click="previewCity(searchResult)">
        {{ searchResult.place_name }}
            </li>
      </template>
    </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList/>
        <template #fallback>
          <CityCardSkeleton/>
        </template>
      </Suspense>
    </div>
  </main>
</template>
