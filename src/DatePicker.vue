<template>
  <div class="date-picker" :style="{
    '--width': `${width}px`
  }">
    <span class="go" @click="backward()">&#8249;</span>

    <ol v-for="offset in offsets" class="calendar">
      <li class="month-name">
        {{ monthName(offset) }} {{ year(offset) }}
      </li>

      <li class="day-name" v-for="name in dayNames">
        {{ name }}
      </li>

      <li class="day-pad" v-for="_ in Array(weekdayPadding(offset))"></li>

      <li class="day"
          :class="Object.assign(
            {
              selected: daySelected(day),
              today: isCurrentDay(day)
            },
            mode === 'range' ? rangeModeCellClasses(day) : {}
          )"
          @mousedown="selected(day)"
          @mouseup="selected(day)"
          @click="selected(day)"
          @mouseover="setLastHovered(day)"
          v-for="day in days(offset)"
      >
        {{ day.getDate() }}
      </li>

      <li class="day-pad" v-if="showExtraBlankRow(offset)" v-for="_ in Array(7)"></li>
    </ol>

    <span class="go" @click="forward()">&#8250;</span>
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
    isEqual,
    differenceInCalendarWeeks,
    subMonths,
    isToday
  } from 'date-fns'

  export default {
    name: 'DatePicker',

    props: {
      value: {
        type: Object,
        default: { first: null, last: null }
      },

      width: {
        type: Number,
        default: 250
      },

      mode: {
        type: String,
        default: 'single',
        validator: v => ['single', 'range'].indexOf(v) > -1
      },

      panels: {
        type: Number,
        default: 1,
        validator: v => v > 0 && v <= 12
      }
    },

    computed: {
      offsets() {
        return Array.from(Array(this.panels).keys())
      }
    },

    methods: {
      isCurrentDay(day) {
        return isToday(day)
      },

      forward() {
        this.reference = addMonths(this.reference, 1)
      },

      backward() {
        this.reference = subMonths(this.reference, 1)
      },

      showExtraBlankRow(offset) {
        return differenceInCalendarWeeks(this.lastDay(offset), this.firstDay(offset)) < 5;
      },

      rangeModeCellClasses(day) {
        return {
          highlighted: this.dayHighlighted(day),
          first: this.isFirst(day),
          last: this.isLast(day),
          'last-highlighted': this.isLastHovered(day),
          'others-highlighted': !this.isLastHovered(day) && this.lastHovered,
          'in-range': this.isInRange(day)
        }
      },

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

      year(offset) {
        return format(addMonths(this.reference, offset), 'YYYY')
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
        const sel = this.selection

        if (!sel.first || this.mode === 'single') {
          sel.first = day
        } else if (isEqual(sel.first, day)) {
          sel.first = null
          sel.last = null
        } else if (isEqual(sel.last, day)) {
          sel.last = null
        } else if (isAfter(day, sel.first)) {
          sel.last = day
        } else {
          sel.first = day
        }

        if (this.mode === 'single') {
          this.$emit('input', sel.first)
        } else {
          this.$emit('input', sel)
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

  .date-picker {
    /*position: relative;*/

    .go {
      display: inline-block;
      border: 0.8px solid #5d5d5d;
      color: #5d5d5d;
      width: 10px;
      margin-top: calc(var(--width) / 7);
      height: calc(var(--width) / 7 * 6 + 10px);
      line-height: calc(var(--width) / 7 * 6 + 10px);
      text-align: center;
      border-radius: 2px;
      vertical-align: top;
    }

    ol {
      --cell-height: calc(var(--width) / 7);

      width: var(--width);
      display: inline-block;
      margin: 0;
      position: relative;

      &:after {
        position: absolute;
        content: '';
        margin-top: calc(var(--width) / 7);
        height: calc(var(--width) / 7 * 6 + 10px);
        width: 1px;
        top: 0;
        right: -0.5px;
        background: #5d5d5d;
      }

      &:last-of-type:after {
        display: none;
      }

      li {
        user-select: none;
        display: inline-block;
        height: var(--cell-height);
        line-height: var(--cell-height);
        position: relative;
        width: calc(100% / 7);
        text-align: center;
        vertical-align: bottom;
        border-radius: 100%;

        &.month-name {
          width: 100%;
        }

        &.day, &.day-pad {
          margin-top: 2px;
        }

        &.day {
          &.today {
            font-weight: bold;
          }

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

          &.highlighted:not(.selected) {
            &:not(.in-range):hover {
              border-radius: 0 100% 100% 0;
            }
          }
        }
      }
    }
  }
</style>