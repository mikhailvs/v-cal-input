<template>
  <div class="demo">
    <div>
      <label for="panels">Panels</label>
      <input id="panels" type="number" v-model.number="panels">
    </div>

    <div>
      <label for="width">Width</label>
      <input id="width" type="number" v-model.number="width">
    </div>

    <div>
      <label for="single">
        <input id="single" type="radio" value="single" v-model="mode">
        Single
      </label>

      <label for="range">
        <input id="range" type="radio" value="range" v-model="mode">
        Range
      </label>
    </div>

    <hr>

    <div>
      <date-picker :panels="panels" :mode="mode" :width="width" v-model="dates">
        <template slot="day" slot-scope="{ day }">
          {{ day }}
        </template>
      </date-picker>
      <div v-if="mode==='single'">dates: {{dates}}</div>
      <div v-if="mode==='range'">dates.first: {{dates.first}}</div>
      <div v-if="mode==='range'">dates.last: {{dates.last}}</div>
    </div>
  </div>
</template>

<script>
  import DatePicker from './DatePicker'

  export default {
    name: 'Demo',

    components: {
      DatePicker
    },

    data() {
      return {
        panels: 1,
        width: 250,
        mode: 'single',
        dates: { first: null, last: null }
      }
    }
  }
</script>

<style lang="scss">
</style>
