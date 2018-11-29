<template>
  <div>
    <ol v-for="offset in [0, 1]">
      <li class="month-name">
        {{ monthName(offset) }}
      </li>

      <li class="day-name" v-for="name in dayNames">
        {{ name }}
      </li>

      <li v-for="_ in Array(weekdayPadding(offset))"></li>

      <li class="day"
          :class="{
            selected: daySelected(day),
            highlighted: dayHighlighted(day),
            first: isFirst(day),
            last: isLast(day),
            'last-highlighted': isLastHovered(day),
            'others-highlighted': !isLastHovered(day) && lastHovered,
            'in-range': isInRange(day)
          }"
          @mousedown="selected(day)"
          @mouseup="selected(day)"
          @click="selected(day)"
          @mouseover="setLastHovered(day)"
          v-for="day in days(offset)"
      >
        {{ day.getDate() }}
      </li>
    </ol>
  </div>
</template>

<script>
  import {
    eachDay,
    startOfMonth,
    endOfMonth,
    format,
    isWithinRange,
    isAfter,
    addMonths,
    isEqual
  } from 'date-fns'

  export default {
    name: 'DatePicker',

    props: [
      'value'
    ],

    methods: {
      setLastHovered(day) {
        if (this.selection.first && isAfter(this.selection.first, day)) {
          this.lastHovered = null

          return
        }

        this.lastHovered = day
      },

      isInRange(day) {
        return this.selection.first &&
          this.selection.last &&
          isWithinRange(day, this.selection.first, this.selection.last)
      },

      isFirst(day) {
        return isEqual(day, this.selection.first);
      },

      isLast(day) {
        return !this.selection.last || isEqual(day, this.selection.last || this.lastHovered);
      },

      isLastHovered(day) {
        return isEqual(day, this.lastHovered);
      },

      weekdayPadding(offset) {
        return this.firstDay(offset).getDay()
      },

      days(offset) {
        return eachDay(this.firstDay(offset), this.lastDay(offset))
      },

      firstDay(offset) {
        return startOfMonth(addMonths(this.reference, offset))
      },

      lastDay(offset) {
        return endOfMonth(addMonths(this.reference, offset))
      },

      monthName(offset) {
        return format(addMonths(this.reference, offset), 'MMMM')
      },

      daySelected(day) {
        return isEqual(this.selection.first, day) ||
          isEqual(this.selection.last, day)
      },

      dayHighlighted(day) {
        if (!this.selection.first) {
          return false
        }

        const first = this.selection.first
        const last = this.selection.last || this.lastHovered

        if (isAfter(first, last)) {
          return false
        }

        return isWithinRange(day, first, last)
      },

      selected(day) {
        if (!this.selection.first) {
          this.selection.first = day

          return
        }

        if (isEqual(this.selection.first, day)) {
          this.selection.first = null
          this.selection.last = null

          return
        }

        if (isEqual(this.selection.last, day)) {
          this.selection.last = null

          return
        }

        if (isAfter(day, this.selection.first)) {
          this.selection.last = day
        } else {
          this.selection.first = day
        }
      }
    },

    data() {
      return {
        reference: new Date(),
        dayNames: ['S', 'M', 'T', 'W', 'T', 'F', 'S'],
        selection: {
          first: null,
          last: null
        },
        lastHovered: null
      }
    }
  }
</script>

<style lang="scss" scoped>
  * {
    box-sizing: border-box;
  }

  ol {
    border: 1px solid black;
    width: 350px;

    li {
      user-select: none;
      display: inline-block;
      height: 50px;
      line-height: 50px;
      position: relative;
      width: calc(100% / 7);
      text-align: center;
      vertical-align: bottom;
      border-radius: 100%;

      &.month-name {
        width: 100%;
        height: 30px;
        line-height: 30px;
      }

      &.day-name {
        height: 30px;
        line-height: 30px;
      }

      &.day {
        margin-top: 2px;

        &:hover {
          background: white;
          color: black;
          z-index: 2;
        }

        &.selected {
          color: white;
          z-index: 1;

          &:before {
            content: '';
            display: block;
            position: absolute;
            background: #4285f4;
            height: 100%;
            width: 100%;
            top: 0;
            left: 0;
            z-index: -1;
            border-radius: 100%;
          }

          &:after {
            content: '';
            display: block;
            position: absolute;
            background: #e8faff;
            height: 100%;
            width: 50%;
            top: 0;
            z-index: -2;
            border-radius: 0;
          }

          &.last:not(.first):after {
            left: 0;
          }

          &.first:after {
            right: 0;
          }

          &.first.last:not(.others-highlighted):after {
            display: none;
          }
        }

        &.highlighted:not(.selected) {
          background: #e8faff;
          border-radius: 0;
        }

        &:not(.selected):hover {
          color: white;

          &:before {
            content: '';
            display: block;
            position: absolute;
            background: #4285f4;
            height: calc(100% - 14px);
            width: calc(100% - 14px);
            top: 7px;
            left: 7px;
            z-index: -3;
            border-radius: 100%;
          }
        }

        &.highlighted:not(.selected) {
          &:not(.in-range):hover {
            border-radius: 0 100% 100% 0;
          }
        }
      }
    }
  }
</style>