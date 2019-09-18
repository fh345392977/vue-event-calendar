<template>
  <div class="cal-wrapper">
    <div class="cal-header">
      <div class="l" @click="preMonth"><div class="arrow-left icon">&nbsp;</div></div>
      <div class="title">{{curYearMonth}}</div>
      <div class="r" @click="nextMonth"><div class="arrow-right icon">&nbsp;</div></div>
    </div>
    <div class="cal-body">
      <div class="weeks">
        <span
          v-for="(dayName, dayIndex) in i18n[calendar.options.locale].dayNames"
          class="item"
          :key="dayIndex"
          >
          {{i18n[calendar.options.locale].dayNames[(dayIndex + calendar.options.weekStartOn) % 7]}}
        </span>
      </div>
      <div class="dates" >
        <div v-for="date in dayList" class="item"
          :class="[{
            today: date.status ? (today == date.date) : false,
            event: date.status ? (date.event != undefined) : false,
            [calendar.options.className] : (date.date == selectedDay)
          }, ...date.customClass]"
          :key="date.date"
          >
          <p class="date-num"
            @click="handleChangeCurday(date)"
            :style="{color: date.event != undefined ? ((date.date == selectedDay) ? '#fff' : customColor) : 'inherit'}">
            {{date.status ? date.date.split(divider)[2] : '&nbsp;'}}</p>
          <span v-if="date.status ? (today == date.date) : false" class="is-today" :style="{backgroundColor: customColor }" ></span>
          <span v-if="date.status ? (date.event != undefined) : false" class="is-event"
            :style="{borderColor: customColor, backgroundColor: (date.date == selectedDay) ? customColor : 'inherit'}"></span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import i18n from '../i18n.js'
import { dateTimeFormatter, isEqualDateStr} from '../tools.js'

const inBrowser = typeof window !== 'undefined'
export default {
  name: 'cal-panel',
  data () {
    return {
      i18n
    }
  },
  props: {
    events: {
      type: Array,
      required: true
    },
    calendar: {
      type: Object,
      required: true
    },
    selectedDay: {
      type: String,
      required: false
    },
    divider: {
      type: String,
      default: '-'
    }
  },
  computed: {
    dayList () {
      let firstDay = new Date(this.calendar.params.curYear, this.calendar.params.curMonth, 1)
      let dayOfWeek = firstDay.getDay()
      // 根据当前日期计算偏移量 // Calculate the offset based on the current date
      if (this.calendar.options.weekStartOn > dayOfWeek) {
        dayOfWeek = dayOfWeek - this.calendar.options.weekStartOn + 7
      } else {
        dayOfWeek = dayOfWeek - this.calendar.options.weekStartOn
      }

      let startDate = new Date(firstDay)
      startDate.setDate(firstDay.getDate() - dayOfWeek)

      let item, status, tempArr = [], tempItem
      for (let i = 0 ; i < 42 ; i++) {
          item = new Date(startDate);
          item.setDate(startDate.getDate() + i);

          if (this.calendar.params.curMonth === item.getMonth()) {
            status = 1
          } else {
            status = 0
          }
          tempItem = {
            date: `${item.getFullYear()}${this.divider}${item.getMonth()+1}${this.divider}${item.getDate()}`,
            status: status,
            customClass: []
          }
          this.events.forEach((event) => {
            if (isEqualDateStr(event.date, tempItem.date)) {
              tempItem.event = true;
            }
          })
          tempArr.push(tempItem)
      }
      return tempArr
    },
    today () {
      let dateObj = new Date()
      return `${dateObj.getFullYear()}${this.divider}${dateObj.getMonth()+1}${this.divider}${dateObj.getDate()}`
    },
    curYearMonth () {
      let tempDate = Date.parse(new Date(`${this.calendar.params.curYear}${this.divider}${this.calendar.params.curMonth+1}${this.divider}01`))
      return dateTimeFormatter(tempDate, this.i18n[this.calendar.options.locale].format)
    },
    customColor () {
      return this.calendar.options.color
    }
  },
  methods: {
    nextMonth () {
      this.$EventCalendar.nextMonth()
      this.$emit('month-changed', this.curYearMonth)
    },
    preMonth () {
      this.$EventCalendar.preMonth()
      this.$emit('month-changed', this.curYearMonth)
    },
    handleChangeCurday (date) {
      if (date.status) {
        this.$emit('cur-day-changed', date.date)
      }
    }
  }
}
</script>
