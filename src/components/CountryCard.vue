<template>
  <div class="card">
    <h2 class="name">{{ country.flag }}&nbsp;{{ country.name.common }}</h2>
    <p class="capital" v-if="country.capital">
      Capital: {{ country.capital[0] }}
    </p>
    <p class="population">
      Population: {{ country.population.toLocaleString() }}
    </p>
    <p class="borders" v-if="formattedBorders">
      Borders: {{ formattedBorders }}
    </p>
    <button @click="focusMapOnThisCountry" class="focusBtn">Focus</button>
  </div>
</template>

<script>
import { EventBus } from "../assets/eventBus";

export default {
  props: {
    country: {
      type: Object,
      required: true,
    },
    allCountries: {
      type: Array,
      required: true,
    },
  },
  computed: {
    formattedBorders() {
      const countryAbbreviationMapping = this.buildCountryAbbreviationMapping(
        this.allCountries
      );
      return this.country.borders
        ?.map((abbreviation) => countryAbbreviationMapping[abbreviation])
        .join(", ");
    },
  },
  methods: {
    focusMapOnThisCountry() {
      const newURL = `/${this.country.name.common}`;
      window.history.pushState(null, null, newURL);
      EventBus.$emit("focus-country", this.country);
    },

    buildCountryAbbreviationMapping(countries) {
      return countries.reduce((mapping, country) => {
        mapping[country.cca3] = country.name.common;
        return mapping;
      }, {});
    },
  },
};
</script>
