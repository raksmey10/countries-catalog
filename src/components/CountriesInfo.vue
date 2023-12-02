<script setup lang="ts">
  defineProps({
    countryInfo: {
      type: Object,
      required: true
    }
  })
</script>

<template>
<!--  <pre>{{ countryInfo }}</pre>-->
<!--  {{ countryInfo }}-->

  <table class="tbl-info">
    <tr>
      <td class="bold-text">Common Name:</td>
      <td>{{ countryInfo.name.common }}</td>
    </tr>

    <tr>
      <td class="bold-text">Capital:</td>
      <td>{{ countryInfo.capital[0] }}</td>
    </tr>

    <tr>
      <td class="bold-text">Timezone:</td>
      <td>{{ countryInfo.timezones[0] }}</td>
    </tr>

    <tr>
      <td class="bold-text">Population:</td>
      <td>{{ countryInfo.population.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",") }}</td>
    </tr>

    <tr>
      <td class="bold-text">Independent:</td>
      <td>
        <span v-if="countryInfo.independent == true">
          <el-tag type="success">Yes</el-tag>
        </span>
        <span v-else>
          <el-tag type="danger">No</el-tag>
        </span>
      </td>
    </tr>

    <tr>
      <td class="bold-text">Continent:</td>
      <td>{{ countryInfo.continents[0] }}</td>
    </tr>

    <tr>
      <td class="bold-text">Region:</td>
      <td>{{ countryInfo.region }}</td>
    </tr>

    <tr>
      <td class="bold-text">Sub Region:</td>
      <td>{{ countryInfo.subregion }}</td>
    </tr>
  </table>
</template>

<style lang="scss" scoped>
 table.tbl-info {
   border-collapse:separate;
   border-spacing: 0 1em;

   td.bold-text{
     font-weight: bold;
     padding-right: 1em;
   }
 }
</style>
