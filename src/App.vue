<template>
  <div id="app">
    <Controls
      :filteredCountries="filteredCountries"
      @search-updated="updateSearch"
      @sort-name="sortCountriesByName"
      @sort-population="sortCountriesByPopulation"
    />
    <CountryList
      :countries="filteredCountries"
      @update:countries="updateCountries"
    />
    <Map :countries="filteredCountries" />
  </div>
</template>

<script>
import Controls from "./components/Controls";
import CountryList from "./components/CountryList";
import Map from "./components/Map";
import "./assets/style.css";

export default {
  name: "App",
  components: {
    Controls,
    CountryList,
    Map,
  },
  data() {
    return {
      countries: [],
      allCountries: [],
      search: "",
    };
  },
  computed: {
    filteredCountries() {
      const filtered = this.countries.filter((country) =>
        country.name.common.toLowerCase().includes(this.search.toLowerCase())
      );
      return filtered;
    },
  },
  methods: {
    updateSearch(newSearch) {
      this.search = newSearch;
    },
    updateCountries({ filtered, all }) {
      this.countries = filtered;
      this.allCountries = all;
    },
    sortCountriesByName(isAscendingName) {
      this.countries.sort((a, b) =>
        isAscendingName
          ? a.name.common.localeCompare(b.name.common)
          : b.name.common.localeCompare(a.name.common)
      );
    },
    sortCountriesByPopulation(isAscendingPopulation) {
      this.countries.sort((a, b) =>
        isAscendingPopulation
          ? a.population - b.population
          : b.population - a.population
      );
    },
  },
};
</script>
