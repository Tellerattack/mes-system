/*
 * @Author: xgw 
 * @Date: 2019-03-20 15:51:48 
 * @Last Modified by: xgw
 * @Last Modified time: 2019-03-20 20:39:23
 */
<!--日历的主体部分，以及接受外部事件以及参数的接口，还有暴漏出去的接口-->
<template>
  <div class="comp-full-calendar">
    <!-- header pick month -->
    <fc-header :current-month="currentMonth" :first-day="firstDay" :locale="locale" @change="emitChangeMonth">

      <div slot="header-left">
        <slot name="fc-header-left">
        </slot>
      </div>

      <div slot="header-right">
        <slot name="fc-header-right">
        </slot>
      </div>
    </fc-header>
    <!-- body display date day and events -->
    <div class="full-calendar-body">
      <!-- 上方显示的星期显示表 -->
      <div class="weeks">
        <span class="week" v-for="(dayIndex,index) in 7" :key="index" :class="{'isBgray': dayIndex == 1 || dayIndex == 7}">{{ (dayIndex - 1) | localeWeekDay(firstDay, locale) }}</span>
      </div>
      <div class="dates" ref="dates">
        <!-- <div class="dates-bg">
          <div class="week-row" v-for="(week,index) in currentDates" :key="index">
            <div class="day-cell" v-for="(day,index) in week" :key="index" :class="{'isBgray': day.weekDay == 0 || day.weekDay == 6}">
              <span class="CircleData FirstGreen"></span>
              <p class="day-number" :class="{'today' : day.isToday,
              'not-cur-month' : !day.isCurMonth,'isCgray' : day.weekDay == 0 || day.weekDay == 6}">{{ day.monthDay }}</p>
            </div>
          </div>
        </div> -->
        <!-- absolute so we can make dynamic td -->
        <div class="dates-events">
          <div class="events-week" v-for="(week,index) in currentDates" :key="index">
            <div class="events-day" v-for="(day,index) in week" :key="index" track-by="$index" @click.stop="dayClick(day.date, $event)" @mouseenter="Dayenter(day.date, $event)" @mouseleave="Dayleave(day.date, $event)">
              <span class="CircleData FirstGreen"></span>
              <p class="day-number" :class="{'today' : day.isToday,
              'not-cur-month' : !day.isCurMonth,'isCgray' : day.weekDay == 0 || day.weekDay == 6}">{{day.monthDay}}</p>
              <!-- 事件的显示框 -->
              <div class="event-box">
                <event-card :event="event" :date="day.date" :firstDay="firstDay" v-for="event in day.events" v-show="event.cellIndex <= eventLimit" @click="eventClick" :key="event">
                  <template slot-scope="p">
                    <slot name="fc-event-card" :event="p.event"></slot>
                  </template>
                </event-card>
                <!-- <p class="more-link" @click.stop="selectThisDay(day, $event)">
                  +
                </p> -->
                <p v-if="day.events.length > eventLimit" class="more-link" @click.stop="selectThisDay(day, $event)">
                  + {{day.events[day.events.length -1].cellIndex - eventLimit}} more
                </p>
              </div>
              <!-- 右侧的显示框 -->
              <div id="RightAlert">
                <div id="Triangle"></div>
                <div id="DeleteTop" @click.stop="deletetip($event)"></div>
              </div>
            </div>
          </div>
        </div>

        <!-- full events when click show more -->
        <div class="more-events" v-show="showMore" :style="{left: morePos.left + 'px', top: morePos.top + 'px'}">
          <div class="more-header">
            <span class="title">{{ moreTitle(selectDay.date) }}</span>
            <span class="close" @click.stop="showMore = false">x</span>
          </div>
          <div class="more-body">
            <ul class="body-list">
              <li v-for="(event,index) in selectDay.events" :key="index" v-show="event.isShow" class="body-item" @click="eventClick(event, $event)">
                {{event.title}}
              </li>
            </ul>
          </div>
        </div>

        <slot name="body-card">

        </slot>

      </div>
    </div>
  </div>
</template>
<script >
import moment from "moment";
import EventCard from "@/components/calendarHome/eventCard.vue";
import Header from "@/components/calendarHome/header.vue";
let dateFunc = {
  getMonthViewStartDate(date, firstDay) {
    firstDay = parseInt(firstDay);
    let start = moment(date);
    let startOfMonth = moment(start.startOf("month"));

    start.subtract(startOfMonth.day(), "days");

    if (startOfMonth.day() < firstDay) {
      start.subtract(7, "days");
    }

    start.add(firstDay, "days");

    return start;
  },
  getMonthViewEndDate(date) {
    return this.getMonthViewStartDate().add(6, "weeks");
  }
};

export default {
  props: {
    events: {
      // events will be displayed on calendar
      type: Array,
      default: []
    },
    locale: {
      type: String,
      default: "zh"
    },
    firstDay: {
      type: Number | String,
      validator(val) {
        let res = parseInt(val);
        return res >= 0 && res <= 6;
      },
      default: 0,
      lang: {
        type: String,
        default: "zh"
      }
    }
  },
  components: {
    "event-card": EventCard,
    "fc-header": Header
  },
  mounted() {
    this.emitChangeMonth(this.currentMonth);
  },
  data() {
    return {
      currentMonth: moment().startOf("month"),
      isLismit: true,
      eventLimit: 3,
      showMore: false,
      morePos: {
        top: 0,
        left: 0
      },
      selectDay: {}
    };
  },
  computed: {
    currentDates() {
      return this.getCalendar();
    }
  },
  methods: {
    deletetip(jsevent) {
      console.log(jsevent.target.parentNode);
      jsevent.target.parentNode.style.display = "none";
    },
    emitChangeMonth(firstDayOfMonth) {
      this.currentMonth = firstDayOfMonth;

      let start = dateFunc.getMonthViewStartDate(
        firstDayOfMonth,
        this.firstDay
      );
      let end = dateFunc.getMonthViewEndDate(firstDayOfMonth, this.firstDay);

      this.$emit("changeMonth", start, end, firstDayOfMonth);
    },
    moreTitle(date) {
      if (!date) return "";
      return moment(date).format("ll");
    },
    getCalendar() {
      // calculate 2d-array of each month
      let monthViewStartDate = dateFunc.getMonthViewStartDate(
        this.currentMonth,
        this.firstDay
      );
      let calendar = [];

      for (let perWeek = 0; perWeek < 6; perWeek++) {
        let week = [];

        for (let perDay = 0; perDay < 7; perDay++) {
          week.push({
            monthDay: monthViewStartDate.date(),
            isToday: monthViewStartDate.isSame(moment(), "day"),
            isCurMonth: monthViewStartDate.isSame(this.currentMonth, "month"),
            weekDay: perDay,
            date: moment(monthViewStartDate),
            events: this.slotEvents(monthViewStartDate)
          });

          monthViewStartDate.add(1, "day");
        }

        calendar.push(week);
      }

      return calendar;
    },
    slotEvents(date) {
      // find all events start from this date
      let cellIndexArr = [];
      let thisDayEvents = this.events.filter(day => {
        let st = moment(day.start);
        let ed = moment(day.end ? day.end : st);

        return date.isBetween(st, ed, null, "[]");
      });

      // sort by duration
      thisDayEvents.sort((a, b) => {
        if (!a.cellIndex) return 1;
        if (!b.cellIndex) return -1;
        return a.cellIndex - b.cellIndex;
      });

      // mark cellIndex and place holder
      for (let i = 0; i < thisDayEvents.length; i++) {
        thisDayEvents[i].cellIndex = thisDayEvents[i].cellIndex || i + 1;
        thisDayEvents[i].isShow = true;
        if (thisDayEvents[i].cellIndex == i + 1 || i > 2) continue;
        thisDayEvents.splice(i, 0, {
          title: "holder",
          cellIndex: i + 1,
          start: date.format(),
          end: date.format(),
          isShow: false
        });
      }

      return thisDayEvents;
    },
    selectThisDay(day, jsEvent) {
      this.selectDay = day;
      this.showMore = true;
      this.morePos = this.computePos(event.target);
      this.morePos.top -= 100;
      let events = day.events.filter(item => {
        return item.isShow == true;
      });
      this.$emit("moreClick", day.date, events, jsEvent);
    },
    computePos(target) {
      let eventRect = target.getBoundingClientRect();
      let pageRect = this.$refs.dates.getBoundingClientRect();
      return {
        left: eventRect.left - pageRect.left,
        top: eventRect.top + eventRect.height - pageRect.top
      };
    },
    dayClick(day, jsEvent) {
      this.$emit("dayClick", day, jsEvent);
    },
    Dayenter(day, jsEvent) {
      this.$emit("DayenterFather", day, jsEvent);
    },
    Dayleave(day, jsEvent) {
      this.$emit("DayleaveFather", day, jsEvent);
    },
    eventClick(event, jsEvent) {
      if (!event.isShow) return;

      jsEvent.stopPropagation();
      let pos = this.computePos(jsEvent.target);
      this.$emit("eventClick", event, jsEvent, pos);
    }
  },
  filters: {
    localeWeekDay(weekday, firstDay, locale) {
      firstDay = parseInt(firstDay);
      const localMoment = moment().locale(locale);
      return localMoment.localeData().weekdaysShort()[(weekday + firstDay) % 7];
    }
  }
};
</script>
<style lang="scss">
.isBgray {
  background-color: #eeeeee;
  color: gray;
}
.isCgray {
  color: gray;
}
#RightAlert {
  width: 150px;
  height: 180px;
  border: 1px solid blue;
  background-color: #f9a101;
  position: absolute;
  top: 60px;
  left: 144px;
  z-index: 999999;
  #Triangle {
    width: 0;
    height: 0;
    border-top: 10px solid transparent;
    border-right: 15px solid #f9a101;
    border-bottom: 10px solid transparent;
    position: absolute;
    top: 10px;
    left: -15px;
  }
  #DeleteTop {
    width: 30px;
    height: 30px;
    border: 1px solid blue;
    position: absolute;
    top: 0px;
    right: 0px;
    background-color: red;
  }
}
.comp-full-calendar {
  // font-family: "elvetica neue", tahoma, "hiragino sans gb";
  padding: 20px;
  // background: #fff;
  max-width: 960px;
  margin: 0 auto;
  ul,
  p {
    margin: 0;
    padding: 0;
  }
}

.full-calendar-body {
  margin-top: 20px;
  .weeks {
    display: flex;
    border-top: 1px solid #e0e0e0;
    border-bottom: 1px solid #e0e0e0;
    border-left: 1px solid #e0e0e0;
    .week {
      flex: 1;
      text-align: center;
      border-right: 1px solid #e0e0e0;
    }
  }
  .dates {
    position: relative;
    .week-row {
      // width: 100%;
      // position:absolute;
      border-left: 1px solid #e0e0e0;
      display: flex;
      .day-cell {
        flex: 1;
        min-height: 112px;
        padding: 4px;
        border-right: 1px solid #e0e0e0;
        border-bottom: 1px solid #e0e0e0;
        .day-number {
          text-align: right;
          // text-align: center;
          // width: 25px;
          // height: 25px;
          // line-height: 25px;
          // float: right;
          // border-radius: 50%;
        }
        &.today {
          background-color: red;
          color: #fff;
          z-index: 99999;
          box-shadow: 2px 2px 2px 2px #888888;
        }
        &.not-cur-month {
          .day-number {
            color: rgba(0, 0, 0, 0.24);
          }
        }
      }
    }
    .dates-events {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 1;
      width: 100%;
      background-color: #fff;
      .events-week {
        display: flex;
        
        .events-day {
          cursor: pointer;
          margin-right:-1px; 
          margin-bottom:-1px;
          border: 1px solid #aaa;
          // border-bottom: 1px solid #aaa;
          // border-left: 1px solid #aaa;
          flex: 1;
          padding: 4px;
          min-height: 112px;
          overflow: hidden;
          text-overflow: ellipsis;
          position: relative;
          .day-number {
            text-align: center;
            width: 25px;
            height: 25px;
            line-height: 25px;
            float: right;
            border-radius: 50%;
          }
          .today {
            background-color: red;
            color: #fff;
            z-index: 99999;
          }
          &.not-cur-month {
            .day-number {
              color: rgba(0, 0, 0, 0.24);
            }
          }
          .event-box {
            // border: 1px solid red;
            padding-top: 26px;
            .event-item {
              cursor: pointer;
              font-size: 12px;
              background-color: #00c561;
              margin-bottom: 2px;
              color: #fff;
              padding: 0 0 0 4px;
              height: 18px;
              line-height: 18px;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
              &.is-start {
                margin-left: 4px;
                border-top-left-radius:4px;
                border-bottom-left-radius:4px;
              }
              &.is-end {
                margin-right: 4px;
                border-top-right-radius:4px;
                border-bottom-right-radius:4px;
              }
              &.is-opacity {
                opacity: 0;
              }
            }
            .success{
              background-color: #67C23A;
            }
            .danger{
              background-color: #F56C6C;
            }
            .warning{
              background-color: #E6A23C;
            }
            .info{
              background-color: #909399;
            }
            .more-link {
              cursor: pointer;
              // text-align: right;
              padding-left: 8px;
              padding-right: 2px;
              color: rgba(0, 0, 0, 0.38);
              font-size: 14px;
            }
          }
        }
      }
    }
    .more-events {
      position: absolute;
      width: 150px;
      z-index: 2;
      border: 1px solid #eee;
      box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
      .more-header {
        background-color: #eee;
        padding: 5px;
        display: flex;
        align-items: center;
        font-size: 14px;
        .title {
          flex: 1;
        }
        .close {
          margin-right: 2px;
          cursor: pointer;
          font-size: 16px;
        }
      }
      .more-body {
        height: 146px;
        overflow: hidden;
        .body-list {
          height: 144px;
          padding: 5px;
          overflow: auto;
          background-color: #fff;
          .body-item {
            cursor: pointer;
            font-size: 12px;
            background-color: #c7e6fd;
            margin-bottom: 2px;
            color: rgba(0, 0, 0, 0.87);
            padding: 0 0 0 4px;
            height: 18px;
            line-height: 18px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
          }
        }
      }
    }
  }
  .FirstGreen {
    background: green;
  }
  .FirstYellow {
    background: yellow;
  }
  .FirstRed {
    background: red;
  }
  .CircleData {
    width: 5px;
    height: 5px;
    border-radius: 100%;
    float: left;
    margin-top: 10px;
    margin-left: 10px;
  }
}
</style>