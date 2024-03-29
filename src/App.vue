<template>
  <div class="date-picker green">
    <input :name="name" :value=" dateChosen ? `${year}-${month}-${day}` : ''" type="hidden">
    <div @click="!open ? setDecadeMode() : false" class="date-header">
      <div v-if="!dateChosen && !open">
        <p class="action-text">Click here to pick a date</p>
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
        <p class="action-text">Click to change</p>
      </div>
    </div>
    <div class="date-panel" v-if="open">
      <div v-if="mode === 'dec'">
        <h5 class="action-text">Pick a decade</h5>
        <div class="option-wrapper decade">
          <div :key="century" class="option-group" v-for="(decades, century) in decadeOptionsByCentury">
            <div :class="['option', (i < minDecade || i > maxDecade ? 'disabled' : ''), (decade === i ? 'active' : '')]" :key="i" @click="pickDecade(i)" v-for="i in decades">
              {{i}}'s
            </div>
          </div>

        </div>
        <hr/>
        <span @click="prevDecadePage()" class="button">Prev</span>
        <span @click="nextDecadePage()" class="button">Next</span>
      </div>
      <div v-if="mode === 'y'">
        <h5 class="action-text">Pick a year</h5>
        <div class="option-wrapper year">
          <div :class="['option', (year === i ? 'active' : '')]" :key="i" @click="pickYear(i)" v-for="i in yearOptions">{{i}}</div>
        </div>
        <hr/>
        <span @click="setDecadeMode()" class="button">Back</span>
      </div>
      <div v-if="mode === 'm'">
        <h5 class="action-text">Pick a month</h5>
        <div class="option-wrapper month">
          <div :class="['option', (month === i ? 'active' : '')]" :key="i" @click="pickMonth(i)" v-for="i in monthOptions">{{monthName(i)}}</div>
        </div>
      </div>
      <div v-if="mode === 'd'">
        <h5 class="action-text">Pick a day</h5>
        <div class="option-wrapper day">
          <div :class="['option', (day === i ? 'active' : '')]" :key="i" @click="pickDay(i)" v-for="i in dayOptions">{{ordinalizeNumber(i)}}</div>
        </div>
      </div>
      <hr/>
      <span @click="open = false" class="button">
        {{dateChosen ? "Done" : "Close"}}
      </span>
    </div>
  </div>
</template>

<script lang="ts">
  import {getDaysInMonth} from "date-fns";
  import {enGB} from "date-fns/locale";
  import {computed, defineComponent, Ref, ref} from "vue";
  import {ordinalizeNumber} from "./lib/ordinalizeNumber";

  export default defineComponent({
    setup(props) {
      let open = ref(false);
      let mode = ref("dec");

      let day: Ref<number | null> = ref(null);
      let month: Ref<number | null> = ref(null);
      let year: Ref<number | null> = ref(null);

      let decade: Ref<number | null> = ref(null);
      let decadePage = ref(new Date().getFullYear() - 200);

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

        return new Date(year.value, month.value, day.value || undefined);
      });
      let decadeOptions = computed(() => {
        let arr = [];
        for (let i = decadePage.value; i <= decadePage.value + 200; i += 10) {
          arr.push(i);
        }
        return arr;
      });
      let decadeOptionsByCentury = computed(() => {
        return decadeOptions.value.reduce(
          (accumulator: { [i: number]: number[] }, decadeYear: number) => {
            let century = Math.floor(decadeYear / 100) + 1;
            accumulator[century] = accumulator[century] || [];
            accumulator[century].push(decadeYear);
            return accumulator;
          },
          {},
        );
      });
      let yearOptions = computed(() => {
        let arr: number[] = [];
        if (decade.value == null) {
          return arr;
        }
        for (let i = decade.value; i <= decade.value + 9; i++) {
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

        let daysInSelectedMonth = getDaysInMonth(new Date(year.value!, month.value! - 1));
        if (day.value !== null && daysInSelectedMonth < day.value) {
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
        // Computed
        decadeOptions,
        decadeOptionsByCentury,
        yearOptions,
        monthOptions,
        dayOptions,
        // Methods
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
      name: {
        type: String,
        required: true,
      },
      minDecade: {
        type: Number,
        required: false,
        default: 400,
      },
      maxDecade: {
        type: Number,
        required: false,
        default: Number.POSITIVE_INFINITY,
      },
    },
  });
</script>

<style lang="postcss">

  .date-picker {
    @apply text-center bg-white;

    hr {
      @apply mt-2 mb-2;
    }

    .button {
      @apply border-2 border-solid;
      @apply rounded-lg;
      @apply p-2 m-2;
      @apply inline-block;
      @apply cursor-pointer;
      @apply select-none;

      &:hover {
        @apply border-2 border-solid;
      }
    }

    .date-header {
      @apply rounded-sm shadow cursor-pointer p-2;
      @apply border-b-2 border-solid;

      .action-text {
        @apply inline-block;
        @apply p-1;
        @apply font-semibold;
        @apply rounded border-2 border-solid border-transparent;

        &:hover {
          @apply border-2 border-solid;
        }
      }

      .date-header-item-separator {
        @apply font-bold ;
        @apply px-1 py-2;
        @apply cursor-default;

        @screen md {
          @apply m-2;
        }
      }

      .date-header-item {
        @apply font-bold;
        @apply cursor-pointer;
        @apply rounded;
        @apply border-2 border-solid;
        @apply inline-block;
        @apply text-center;
        @apply px-2 py-1;

        &:hover {
          @apply bg-grey-400 text-grey-400-contrast;
          @apply border-2 border-solid;
        }

        &.enabled {
          @apply border-2 border-solid;

          &:hover {
            @apply bg-white text-white-contrast;
            @apply border-2 border-solid;
          }
        }

        &.active {
          @apply bg-white text-white-contrast;
        }

      }
    }

    .date-panel {
      @apply p-2;
      @apply rounded-sm;
      @apply shadow;

      .action-text {
        @apply font-bold;
        @apply mt-4;
        @apply mb-4;
        @apply text-lg;
      }

      .option-group {
        flex-basis: 100%;
        @apply mb-4;
      }

      .option-wrapper, .option-group {
        @apply mb-8;
        @apply flex flex-wrap content-center justify-center text-center;

        &.decade {
          .option {
            @screen sm {
              flex-basis: 45%;
            }
            @screen md {
              flex-basis: 17%;
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
            @screen sm {
              flex-basis: 45%;
            }
            @screen md {
              flex-basis: 19%;
            }
          }
        }

        &.day {
          .option {
            @screen sm {
              flex-basis: 15%;
            }
            @screen md {
              flex-basis: 15%;
            }
          }
        }

        .option {
          @apply select-none;
          @apply m-1;
          @apply p-1;
          @apply border-2 border-solid;
          @apply rounded;
          @apply cursor-pointer;

          &.disabled {
            @apply cursor-default;
            @apply border-2 border-solid ;
            @apply bg-grey-200 text-grey-200-contrast ;
          }

          &:hover:not(.disabled), &.active {
            @apply bg-white text-white-contrast;
          }
        }
      }
    }

    &.pink {
      .button {
        @apply bg-pink-700 text-pink-700-contrast border-pink-700;

        &:hover {
          @apply bg-pink-200 text-pink-200-contrast border-pink-700;
        }
      }

      .date-header {
        @apply bg-pink-200 text-pink-200-contrast border-pink-500;

        .action-text {
          &:hover {
            @apply bg-pink-300 text-pink-300-contrast border-pink-700;
          }
        }
      }

      .date-header-item {
        @apply bg-pink-700 text-pink-700-contrast border-pink-700;

        &:hover {
          @apply border-pink-700;
        }

        &.enabled {
          @apply bg-pink-700 text-pink-700-contrast border-pink-700;

          &:hover {
            @apply border-pink-700;
          }
        }

        &.active {
          @apply bg-pink-700 text-pink-700-contrast;
        }
      }

      .date-panel {
        .option-wrapper, .option-group {
          .option {
            @apply bg-pink-200 text-pink-200-contrast border-pink-500;

            &.disabled {
              @apply border-pink-300;
            }
          }
        }
      }
    }

    &.blue {
      .button {
        @apply bg-blue-700 text-blue-700-contrast border-blue-700;

        &:hover {
          @apply bg-blue-200 text-blue-200-contrast border-blue-700;
        }
      }

      .date-header {
        @apply bg-blue-200 text-blue-200-contrast border-blue-500;

        .action-text {
          &:hover {
            @apply bg-blue-300 text-blue-300-contrast border-blue-700;
          }
        }
      }

      .date-header-item {
        @apply bg-blue-700 text-blue-700-contrast border-blue-700;

        &:hover {
          @apply border-blue-700;
        }

        &.enabled {
          @apply bg-blue-700 text-blue-700-contrast border-blue-700;

          &:hover {
            @apply border-blue-700;
          }
        }

        &.active {
          @apply bg-blue-700 text-blue-700-contrast;
        }
      }

      .date-panel {
        .option-wrapper, .option-group {
          .option {
            @apply bg-blue-200 text-blue-200-contrast border-blue-500;

            &.disabled {
              @apply border-blue-300;
            }
          }
        }
      }
    }

    &.green {
      .button {
        @apply bg-green-700 text-green-700-contrast border-green-700;

        &:hover {
          @apply bg-green-200 text-green-200-contrast border-green-700;
        }
      }

      .date-header {
        @apply bg-green-200 text-green-200-contrast border-green-500;

        .action-text {
          &:hover {
            @apply bg-green-300 text-green-300-contrast border-green-700;
          }
        }
      }

      .date-header-item {
        @apply bg-green-700 text-green-700-contrast border-green-700;

        &:hover {
          @apply border-green-700;
        }

        &.enabled {
          @apply bg-green-700 text-green-700-contrast border-green-700;

          &:hover {
            @apply border-green-700;
          }
        }

        &.active {
          @apply bg-green-700 text-green-700-contrast;
        }
      }

      .date-panel {
        .option-wrapper, .option-group {
          .option {
            @apply bg-green-200 text-green-200-contrast border-green-500;

            &.disabled {
              @apply border-green-300;
            }
          }
        }
      }
    }

    &.amber {
      .button {
        @apply bg-amber-700 text-amber-700-contrast border-amber-700;

        &:hover {
          @apply bg-amber-200 text-amber-200-contrast border-amber-700;
        }
      }

      .date-header {
        @apply bg-amber-200 text-amber-200-contrast border-amber-500;

        .action-text {
          &:hover {
            @apply bg-amber-300 text-amber-300-contrast border-amber-700;
          }
        }
      }

      .date-header-item {
        @apply bg-amber-700 text-amber-700-contrast border-amber-700;

        &:hover {
          @apply border-amber-700;
        }

        &.enabled {
          @apply bg-amber-700 text-amber-700-contrast border-amber-700;

          &:hover {
            @apply border-amber-700;
          }
        }

        &.active {
          @apply bg-amber-700 text-amber-700-contrast;
        }
      }

      .date-panel {
        .option-wrapper, .option-group {
          .option {
            @apply bg-amber-200 text-amber-200-contrast border-amber-500;

            &.disabled {
              @apply border-amber-300;
            }
          }
        }
      }
    }
  }
</style>
