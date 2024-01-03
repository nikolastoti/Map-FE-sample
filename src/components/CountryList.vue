<template>
  <div class="grid">
    <CountryCard
      v-for="country in countries"
      :key="country.alpha3Code"
      :country="country"
      :allCountries="allCountries"
    />
  </div>
</template>

<script>
import axios from "axios";
import CountryCard from "./CountryCard.vue";

export default {
  props: {
    countries: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      filteredCountries: [],
      countriesData: [],
    };
  },
  components: {
    CountryCard,
  },
  methods: {
    fetchCountries() {
      axios
        .get("https://restcountries.com/v3.1/all")
        .then((response) => {
          this.countriesData = response.data;
          this.allCountries = response.data;
          this.$emit("update:countries", {
            filtered: this.countriesData,
            all: this.allCountries,
          });
        })
        .catch((error) => {
          console.error("Error fetching data: ", error);
        });
    },
  },
  created() {
    this.fetchCountries();
  },
};
</script>
