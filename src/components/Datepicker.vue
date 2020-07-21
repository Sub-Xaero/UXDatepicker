<template>
  <div class="date-picker">
    <input :value=" dateChosen ? `${year}-${month}-${day}` : ''" type="hidden">
    <div @click="!open ? setDecadeMode() : false" class="date-header">
      <div v-if="!dateChosen && !open">
        <p class="marginless">Click here to pick a date</p>
        <hr/>
      </div>
      <div>
        <span :class="['date-header-item', 'enabled', (open && (mode === 'y' || mode === 'dec') ? 'active' : '' ) ]" @click="setDecadeMode()">{{ year || "Year" }}</span>
        <span class="date-header-item-separator">-</span>
        <span :class="['date-header-item', (!year ? '' : 'enabled'), (open && (mode === 'm') ? 'active' : '' ) ]" @click="!year ? setDecadeMode() : setMonthMode()">{{ month ? monthName(month) :
          "Month" }}</span>
        <span class="date-header-item-separator">-</span>
        <span :class="['date-header-item', (!year || !month ? '' : 'enabled'), (open && (mode === 'd') ? 'active' : '' ) ]" @click="!year || !month ? setDecadeMode() : setDayMode()">{{ day ?
          ordinalizeNumber(day) : "Day" }}</span>
      </div>
      <div v-if="dateChosen && !open">
        <hr class="mb-1 mt-1"/>
        <p class="marginless">Click to change</p>
      </div>
    </div>
    <div class="date-panel" v-if="open">
      <div v-if="mode === 'dec'">
        <h5>Pick a decade</h5>
        <div class="option-wrapper decade">
          <div :class="['option', (i < minDecade || i > maxDecade ? 'disabled' : ''), (decade === i ? 'active' : '')]" :key="i" @click="pickDecade(i)" v-for="i in decadeOptions">{{i}}'s</div>
        </div>
        <hr/>
        <span @click="prevDecadePage()" class="button dark-blue small">Prev</span>
        <span @click="nextDecadePage()" class="button dark-blue small">Next</span>
      </div>
      <div v-if="mode === 'y'">
        <h5>Pick a year</h5>
        <div class="option-wrapper year">
          <div :class="['option', (year === i ? 'active' : '')]" :key="i" @click="pickYear(i)" v-for="i in yearOptions">{{i}}</div>
        </div>
      </div>
      <div v-if="mode === 'm'">
        <h5>Pick a month</h5>
        <div class="option-wrapper month">
          <div :class="['option', (month === i ? 'active' : '')]" :key="i" @click="pickMonth(i)" v-for="i in monthOptions">{{monthName(i)}}</div>
        </div>
      </div>
      <div v-if="mode === 'd'">
        <h5>Pick a day</h5>
        <div class="option-wrapper day">
          <div :class="['option', (day === i ? 'active' : '')]" :key="i" @click="pickDay(i)" v-for="i in dayOptions">{{ordinalizeNumber(i)}}</div>
        </div>
      </div>
      <hr/>
      <span @click="open = false" class="button dark-blue small expanded">
        {{dateChosen ? "Done" : "Close"}}
      </span>
    </div>
  </div>
</template>

<script lang="ts">
  import {getDaysInMonth} from "date-fns";
  import {enGB} from "date-fns/locale";
  import {computed, defineComponent, Ref, ref} from "vue";
  import {ordinalizeNumber} from "../lib/ordinalizeNumber";

  export default defineComponent({
    setup(props) {
      let open = ref(false);
      let mode = ref("dec");

      let day: Ref<number | null> = ref(null);
      let month: Ref<number | null> = ref(null);
      let year: Ref<number | null> = ref(null);

      let decade: Ref<number | null> = ref(null);
      let decadePage = ref(1820);

      let {minDecade, maxDecade} = props;

      let dateChosen = computed(() => {
        return !!year.value && !!month.value && !!day.value;
      });
      let date = computed(() => {
        if (year.value == null) {
          return "";
        }
        if (month.value == null) {
          return "";
        }

        return new Date(year.value!, month.value!, day.value!);
      });
      let decadeOptions = computed(() => {
        let arr = [];
        for (let i = decadePage.value; i <= decadePage.value + 200; i += 10) {
          arr.push(i);
        }
        return arr;
      });
      let yearOptions = computed(() => {
        let arr: number[] = [];
        for (let i = decade.value!; i <= decade.value! + 9; i++) {
          arr.push(i);
        }
        return arr;
      });
      let monthOptions = computed(() => {
        let arr = [];
        for (let i = 1; i <= 12; i++) {
          arr.push(i);
        }
        return arr;
      });
      let dayOptions = computed(() => {
        if (year.value === null || month.value === null) {
          return [];
        }

        let arr = [];
        for (let i = 1; i <= getDaysInMonth(new Date(year.value!, month.value! - 1)); i++) {
          arr.push(i);
        }
        return arr;
      });

      let monthName = (monthNumber: number) => {
        return enGB.localize!.month(monthNumber - 1, {});
      };
      let setDayMode = () => {
        mode.value = "d";
        open.value = true;
      };
      let setMonthMode = () => {
        mode.value = "m";
        open.value = true;
      };
      let setYearMode = () => {
        mode.value = "y";
        open.value = true;
      };
      let setDecadeMode = () => {
        mode.value = "dec";
        open.value = true;
      };
      let pickDecade = (newDecade: number) => {
        if (newDecade < minDecade || newDecade > maxDecade) {
          return;
        }
        decade.value = newDecade;
        setYearMode();
      };
      let pickYear = (newYear: number) => {
        year.value = newYear;
        setMonthMode();
      };
      let pickMonth = (newMonth: number) => {
        month.value = newMonth;
        setDayMode();

        let daysInSelectedMonth = getDaysInMonth(new Date(year.value!, month.value - 1));
        if (day.value !== null && daysInSelectedMonth < day.value!) {
          day.value = null;
        }
      };
      let pickDay = (newDay: number) => {
        day.value = newDay;
        open.value = false;
      };

      let prevDecadePage = () => {
        if (decadePage.value <= minDecade) {
          return;
        }

        decadePage.value -= 200;
      };
      let nextDecadePage = () => {
        if (decadePage.value + 200 >= maxDecade) {
          return;
        }
        decadePage.value += 200;
      };


      return {
        // Data
        open,
        mode,
        day,
        month,
        year,
        decade,
        decadePage,
        dateChosen,
        date,
        decadeOptions,
        yearOptions,
        monthOptions,
        dayOptions,
        ordinalizeNumber,
        monthName,
        setDayMode,
        setMonthMode,
        setYearMode,
        setDecadeMode,
        pickDecade,
        pickYear,
        pickMonth,
        pickDay,
        prevDecadePage,
        nextDecadePage,
      };
    },
    props: {
      minDecade: {
        type: Number,
        required: false,
        default: 1400,
      },
      maxDecade: {
        type: Number,
        required: false,
        default: 2200,
      },
    },
  });
</script>

<style>
  /*@import '../stylesheets/shared/foundation_settings';*/
  /*@import '../stylesheets/shared/colors';*/

  .date-picker {
    text-align: center;
    /*!*border: 2px solid palette(brevio, dark-blue);*!*/
    @apply bg-white;

    .date-header {
      @apply bg-blue-900;
      @apply blue-900-contrast;
      @apply p-2;
      @apply cursor-pointer;
      @apply rounded;
      @apply shadow;

      .date-header-item-separator {
        @apply font-bold;

        @screen sm {
          @apply py-1 px-3;
        }
        @screen md {
          @apply py-1 px-3;
          @apply m-1;
        }
      }

      /*.date-header-item {*/
      /*  font-weight: bold;*/
      /*  cursor: pointer;*/
      /*  border-radius: 4px;*/
      /*!*  border: 2px solid palette(brevio, dark-blue);*!*/

      /*  @include breakpoint(small only) {*/
      /*    padding: 0.1em 0.4em;*/
      /*  }*/
      /*  @include breakpoint(medium up) {*/
      /*    padding: 0.1em 0.4em;*/
      /*    margin: 2px;*/
      /*  }*/

      /*  &.enabled:hover {*/
      /*!*    background-color: palette(brevio, dark-blue);*!*/
      /*!*    border: 2px solid palette(brevio, bright-blue);*!*/
      /*  }*/

      /*  &.active {*/
      /*!*    background-color: palette(brevio, bright-blue);*!*/
      /*!*    color: palette(brevio, white);*!*/
      /*  }*/
      /*}*/
    }

    .date-panel {
      /*border-top: 2px solid palette(brevio, dark-blue);*/
      padding: 1em;
      border-radius: 4px;
      box-shadow: 0 0 2px 1px $medium-gray;

      .option-wrapper {
        display: flex;
        flex-wrap: wrap;
        align-content: center;
        justify-content: center;
        text-align: center;

        &.decade {
          .option {
            @include breakpoint(small only) {
              flex-basis: 45%;
            }
            @include breakpoint(medium up) {
              flex-basis: 10%;
            }
          }
        }

        &.year {
          .option {
            flex-basis: 17%;
          }
        }

        &.month {
          .option {
            @include breakpoint(small only) {
              flex-basis: 45%;
            }
            @include breakpoint(medium up) {
              flex-basis: 19%;
            }
          }
        }

        &.day {
          .option {
            @include breakpoint(small only) {
              flex-basis: 15%;
            }
            @include breakpoint(medium up) {
              flex-basis: 15%;
            }
          }
        }

        .option {
          user-select: none;
          margin: 0.25em;
          padding: 0.5em;
          /*border: 2px solid palette(brevio, green);*/
          /*background-color: palette(brevio, light-green);*/
          /*color: palette(brevio, dark-blue);*/
          border-radius: 1em;
          cursor: pointer;

          &.disabled {
            cursor: default;
            /*border: 2px solid palette(brevio, light-grey);*/
            /*background-color: palette(brevio, light-grey);*/
            /*color: palette(brevio, white);*/
          }

          &:hover:not(.disabled), &.active {
            /*!*border: 2px solid palette(brevio, dark-blue);*!*/
            /*background-color: palette(brevio, white);*/
            /*color: palette(brevio, dark-blue);*/
          }
        }
      }
    }
  }
</style>
