<template>
  <div class="controls">
    <a href="https://www.kpler.com/" target="_blank">
      <img id="logo" src="../../public/logo.png" />
    </a>
    <input
      v-model="search"
      type="text"
      class="search-input"
      placeholder="Search countries"
    />
    <button @click="toggleSortOrder('name')">
      <span v-if="isAscendingName">↑</span>
      <span v-else-if="isAscendingName === false">↓</span>
      Sort by Name
    </button>
    <button @click="toggleSortOrder('population')">
      <span v-if="isAscendingPopulation">↑</span>
      <span v-else-if="isAscendingPopulation === false">↓</span>
      Sort by Population
    </button>
    <div class="average-population">
      Average Population: {{ averagePopulation }}
    </div>
  </div>
</template>

<script>
export default {
  props: {
    filteredCountries: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      search: "",
      isAscendingName: undefined,
      isAscendingPopulation: undefined,
    };
  },
  computed: {
    averagePopulation() {
      if (this.filteredCountries.length === 0) return 0;
      const totalPopulation = this.filteredCountries.reduce(
        (acc, country) => acc + country.population,
        0
      );
      return Math.round(
        totalPopulation / this.filteredCountries.length
      ).toLocaleString();
    },
  },
  methods: {
    toggleSortOrder(type) {
      if (type === "name") {
        this.isAscendingName = !this.isAscendingName;
        this.$emit("sort-name", this.isAscendingName);
      } else if (type === "population") {
        this.isAscendingPopulation = !this.isAscendingPopulation;
        this.$emit("sort-population", this.isAscendingPopulation);
      }
    },
  },
  watch: {
    search(newSearch) {
      this.$emit("search-updated", newSearch);
    },
  },
};
</script>
