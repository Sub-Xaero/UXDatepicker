<template>
  <div class="date-picker text-center border-2 border-blue-900 bg-white">
    <input :value=" dateChosen ? `${year}-${month}-${day}` : ''" type="hidden">
    <div @click="!open ? setDecadeMode() : false" class=" bg-blue-200 blue-200-contrast p-2 cursor-pointer rounded shadow;">
      <div v-if="!dateChosen && !open">
        <p class="marginless">Click here to pick a date</p>
        <hr/>
      </div>
      <div>
        <span :class="[
          'date-header-item font-bold cursor-pointer rounded border-2 border-blue-900 py-1 px-3 md:m-1 inline-flex',
          'hover:bg-blue-900 hover:border-2 hover:border-blue-300',
          (open && (mode === 'y' || mode === 'dec') ? 'active' : '' )
          ]"
              @click="setDecadeMode()">
          {{ year || "Year" }}
        </span>

        <span class="font-bold py-1 px-3 md:m-1">-</span>

        <span :class="[
          'date-header-item font-bold cursor-pointer rounded border-2 border-blue-900 py-1 px-3 md:m-1 inline-flex',
          (!year ? '' : 'hover:bg-blue-900 hover:border-2 hover:border-blue-300'),
          (open && (mode === 'm') ? 'active' : '' )
          ]"
              @click="!year ? setDecadeMode() : setMonthMode()">
          {{ month ? monthName(month) : "Month" }}
        </span>

        <span class="font-bold py-1 px-3 md:m-1">-</span>
        <span :class="[
          'date-header-item font-bold cursor-pointer rounded border-2 border-blue-900 py-1 px-3 md:m-1 inline-flex',
          (!year || !month ? '' : 'hover:bg-blue-900 hover:border-2 hover:border-blue-300'),
          (open && (mode === 'd') ? 'active' : '' )
          ]"
              @click="!year || !month ? setDecadeMode() : setDayMode()">
          {{ day ? ordinalizeNumber(day) : "Day" }}
        </span>
      </div>
      <div v-if="dateChosen && !open">
        <hr class="mb-1 mt-1"/>
        <p class="marginless">Click to change</p>
      </div>
    </div>
    <div
      :class="[
        'date-panel border-t-2 border-blue-900 p-1 rounded shadow',
        ]"
      v-if="open"
    >
      <div v-if="mode === 'dec'">
        <h5 class="text-lg font-bold">Pick a decade</h5>
        <div
          :class="[
          'option-wrapper flex flex-wrap content-center justify-center text-center',
           'decade'
           ]"
        >
          <div
            :class="[
            'option select-none m-1 p-1 border-2 border-green-500 bg-green-200 text-green-200-contrast rounded cursor-pointer',
            'hover:border-2 hover:border-blue-900 hover:bg-white hover:text-white-contrast ',
            (decade === i ? 'active border-2 border-blue-900 bg-white text-white-contrast' : ''),,
            (i < minDecade || i > maxDecade ? 'disabled cursor-default border-2 border-grey-200 bg-grey-200 text-grey-200-contrast' : ''),
            ]"
            :key="i"
            @click="pickDecade(i)"
            v-for="i in decadeOptions"
          >
            {{i}}'s
          </div>
        </div>
        <hr/>
        <span @click="prevDecadePage()" class="rounded bg-blue-900 text-blue-900-contrast inline-block p-2 m-1 hover:bg-blue-400 hover:text-blue-400-contrast">Prev</span>
        <span @click="nextDecadePage()" class="rounded bg-blue-900 text-blue-900-contrast inline-block p-2 m-1 hover:bg-blue-400 hover:text-blue-400-contrast">Next</span>
      </div>
      <div v-if="mode === 'y'">
        <h5 class="text-lg font-bold">Pick a year</h5>
        <div
          :class="[
          'option-wrapper flex flex-wrap content-center justify-center text-center',
           'year',
           ]"
        >
          <div
            :class="[
            'option select-none m-1 p-1 border-2 border-green-500 bg-green-200 text-green-200-contrast rounded cursor-pointer',
            'hover:border-2 hover:border-blue-900 hover:bg-white hover:text-white-contrast ',
            (year === i ? 'active border-2 border-blue-900 bg-white text-white-contrast' : ''),
            ]"
            :key="i"
            @click="pickYear(i)"
            v-for="i in yearOptions"
          >
            {{i}}
          </div>
        </div>
      </div>
      <div v-if="mode === 'm'">
        <h5 class="text-lg font-bold">Pick a month</h5>
        <div
          :class="[
          'option-wrapper flex flex-wrap content-center justify-center text-center',
          'month',
           ]"
        >
          <div
            :class="[
            'option select-none m-1 p-1 border-2 border-green-500 bg-green-200 text-green-200-contrast rounded cursor-pointer',
            'hover:border-2 hover:border-blue-900 hover:bg-white hover:text-white-contrast ',
            (month === i ? 'active border-2 border-blue-900 bg-white text-white-contrast' : ''),
            ]"
            :key="i"
            @click="pickMonth(i)"
            v-for="i in monthOptions"
          >
            {{monthName(i)}}
          </div>
        </div>
      </div>
      <div v-if="mode === 'd'">
        <h5 class="text-lg font-bold">Pick a day</h5>
        <div
          :class="[
          'option-wrapper flex flex-wrap content-center justify-center text-center',
          'day',
         ]"
        >
          <div
            :class="[
              'option select-none m-1 p-1 border-2 border-green-500 bg-green-200 text-green-200-contrast rounded cursor-pointer',
              'hover:border-2 hover:border-blue-900 hover:bg-white hover:text-white-contrast ',
              (day === i ? 'active border-2 border-blue-900 bg-white text-white-contrast' : ''),
              ]"
            :key="i"
            @click="pickDay(i)"
            v-for="i in dayOptions"
          >
            {{ordinalizeNumber(i)}}
          </div>
        </div>
      </div>
      <hr/>
      <span @click="open = false" class="rounded bg-blue-900 text-blue-900-contrast inline-block p-2 m-1 hover:bg-blue-400 hover:text-blue-400-contrast">
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
        // Computed
        decadeOptions,
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
    .date-panel {
      .option-wrapper {

        &.decade {
          .option {
            @screen sm {
              flex-basis: 45%;
            }
            @screen md {
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

      }
    }
  }
</style>
