<template>
  <div class="calendar">

    <div class="toolbar">
      <div class="current-date">{{ monthName }}, {{ currentYear }}</div>
      <div class="nav">
        <div class="item" @click="previousMonth">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 512">
            <path
              d="M9.4 233.4c-12.5 12.5-12.5 32.8 0 45.3l160 160c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L77.3 256 214.6 118.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0l-160 160z"
            />
          </svg>
        </div>
        <div class="item" @click="nextMonth">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 512">
            <path
              d="M246.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-160 160c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L178.7 256 41.4 118.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l160 160z"
            />
          </svg>
        </div>
      </div>
    </div>

    <div class="week-days">
      <div class="day" v-for="day in weekDays" :key="day">{{ day }}</div>
    </div>

    <div class="grid">
      <div
        class="day other-days"
        v-for="day in firstMonthDay - 1"
        :key="'prev' + day"
      >
        {{ prevMonthDays + 1 - (firstMonthDay - day) }}
      </div>
      <div
        class="day"
        :class="[
          {
            current:
              day === currentTime.day &&
              currentMonth === currentTime.month &&
              currentYear === currentTime.year,
          },
          {
            event:
              day === eventDate.day &&
              currentMonth === eventDate.month &&
              currentYear === eventDate.year,
          },
        ]"
        v-for="(day, index) in currentDaysMonth"
        :key="index"
      >
        {{ day }}
        <div
          class="event-time"
          v-if="
            day === eventDate.day &&
            currentMonth === eventDate.month &&
            currentYear === eventDate.year
          "
        >
          {{ eventDate.startTime }} - {{ eventDate.eventTimeEnd }}
        </div>
      </div>
      <div
        class="day other-days"
        v-for="day in nextOtherDays - (currentDaysMonth + firstMonthDay)"
        :key="'next' + day"
      >
        {{ day }}
      </div>
    </div>

  </div>
</template>

<script setup>
import getDay from "date-fns/getDay";
import getDaysInMonth from "date-fns/getDaysInMonth";
import { zonedTimeToUtc, utcToZonedTime } from "date-fns-tz";

import { computed, ref } from "vue";

const weekDays = ["Пн", "Вт", "Сп", "Чт", "Пт", "Сб", "Нд"];

const today = new Date(
  new Date().toLocaleString("en", { timeZone: "Europe/London" })
);

//converting one timezone to another
const eventTimeZoned = zonedTimeToUtc("2023-03-26 14:00", "Europe/Kyiv");
const eventTime = utcToZonedTime(eventTimeZoned, "Europe/London");

const eventTimeZonedEnd = zonedTimeToUtc("2023-03-26 15:00", "Europe/Kyiv");
const eventTimeEnd = utcToZonedTime(eventTimeZonedEnd, "Europe/London");

//data to add an event
const eventDate = {
  day: eventTime.getDate(),
  month: eventTime.getMonth(),
  year: eventTime.getFullYear(),
  startTime: eventTime.toLocaleTimeString("en-US", {
    hour12: false,
    hour: "numeric",
    minute: "numeric",
  }),
  eventTimeEnd: eventTimeEnd.toLocaleTimeString("en-US", {
    hour12: false,
    hour: "numeric",
    minute: "numeric",
  }),
};

//needed to highlight today in the calendar
const currentTime = {
  day: today.getDate(),
  month: today.getMonth(),
  year: today.getFullYear(),
};

//selected date
const currentMonth = ref(today.getMonth());
const currentYear = ref(today.getFullYear());

//name of the month(under the calendar)
const monthName = computed(() => {
  return new Date(currentYear.value, currentMonth.value).toLocaleString(
    "default",
    { month: "long" }
  );
});

const currentDaysMonth = computed(() => {
  return getDaysInMonth(new Date(currentYear.value, currentMonth.value));
});

const prevMonthDays = computed(() => {
  let year =
    currentMonth.value === 0 ? currentYear.value - 1 : currentYear.value;
  let month = currentMonth.value === 0 ? 11 : currentMonth.value - 1;
  return getDaysInMonth(new Date(year, month));
});

const firstMonthDay = computed(() => {
  let firstDay = getDay(new Date(currentYear.value, currentMonth.value, 1));
  if (firstDay === 0) firstDay = 7;
  return firstDay;
});

//needed for a beautiful display of the days of the next month
const nextOtherDays = computed(() => {
  if (firstMonthDay.value >= 6) {
    return 43;
  } else {
    return 36;
  }
});

//navigation between months
const previousMonth = () => {
  if (currentMonth.value === 0) {
    currentMonth.value = 11;
    currentYear.value -= 1;
  } else {
    currentMonth.value -= 1;
  }
};

const nextMonth = () => {
  if (currentMonth.value === 11) {
    currentMonth.value = 0;
    currentYear.value++;
  } else {
    currentMonth.value++;
  }
};
</script>

<style lang="scss" scoped>
.calendar {
  .toolbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;

    .current-date {
      font-size: 20px;
      font-weight: 600;
      text-transform: capitalize;
    }

    .nav {
      display: flex;
      gap: 10px;
      user-select: none;

      .item {
        width: 30px;
        height: 30px;
        display: grid;
        place-items: center;
        cursor: pointer;
        border: 2px solid black;
        border-radius: 4px;

        svg {
          width: 10px;
        }
      }
    }
  }

  .week-days {
    display: flex;
    margin-bottom: 15px;
    border-bottom: 1px solid #dbdbdb;

    .day {
      width: 100px;
      padding: 10px;
      font-size: 18px;
      color: #a6a6a6;
    }
  }

  .grid {
    display: flex;
    flex-wrap: wrap;
    max-width: 700px;

    .day {
      height: 100px;
      width: 100px;
      padding: 10px;
      border-right: 1px solid #dbdbdb;
      border-bottom: 1px solid #dbdbdb;

      &.current {
        color: #ffff;
        background-color: #ff5656;
      }

      &.event {
        background-color: #ffffaa;
      }

      &.other-days {
        color: #a6a6a6;
      }

      &:nth-last-child(-n + 7) {
        border-bottom: none;
      }

      &:nth-child(7n) {
        border-right: none;
      }

      .event-time {
        font-size: 12px;
        margin-top: 5px;
      }
    }
  }
}
</style>
