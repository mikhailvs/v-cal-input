# vue-calendar-input
Simple, clean calendar input with single date select, or date range selection.

### [Demo](https://mikhailvs.github.io/vue-calendar-input/)

## Installation

```
yarn add vue-calendar-input
```

or

```
npm install vue-calendar-input
```

## Usage

```vue
<template>
  <div>
    <h1>Selected Date: {{ date }}</h1>
    <date-picker v-model="date"/>
  </div>
</template>

<script>
  import DatePicker from 'vue-calendar-input'

  export default {
    name: 'Demo',
    components: { DatePicker },
    data() {
      return { date: null }
    }
  }
</script>
```

## Configuration
### props
| name | default | description |
| ---- | ------- | ------------|
| value | `null` | the selected date(s) |
| width | `350` | width of the calendar in `px` |
| mode | `'single'` | `'single'` for single date, `'range'` for date ranges |
| panels | `1` | the number of month panels to display |
