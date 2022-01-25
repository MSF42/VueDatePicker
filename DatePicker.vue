<template>
  <div
    class="modal-backdrop opacity-25"
    v-if="showDate"
    @click="toggleDateView()"
  ></div>
  <div class="position-relative">
    <div
      @click="toggleDateView()"
      class="p-1 mx-1 mb-2 h6 text-center bg-gray-100 cal-small"
    >
      {{ dateYear }} - {{ dateMonthStr }} - {{ dateDate }}
    </div>
    <div
      v-if="showDate"
      class="cal-main-box border-blue-700 p-2 border-5 bg-gray-100 position-absolute"
    >
      <div
        class="cal-inner-box border-blue-700 border-1 bg-gray-200 m-auto h-100 w-100"
      >
        <table class="table table-sm" style="table-layout: fixed">
          <thead>
            <tr class="text-center text-blue-700 bg-gray-300 h3">
              <th colspan="2" @click="dateChange(-1, 0)">
                <i class="bi bi-caret-left-fill h4 text-blue-700"></i>
                <i class="bi bi-caret-left-fill h4 text-blue-700"></i>
              </th>
              <th colspan="3">
                {{ dateYear }}
              </th>
              <th colspan="2" @click="dateChange(1, 0)">
                <i class="bi bi-caret-right-fill h4 text-blue-700"></i>
                <i class="bi bi-caret-right-fill h4 text-blue-700"></i>
              </th>
            </tr>
            <tr class="text-center bg-blue-700 text-gray-300">
              <th colspan="1" @click="dateChange(0, -1)">
                <i class="bi bi-caret-left-fill h6 text-gray-300"></i>
              </th>
              <th scope="col" colspan="5" class="h4">
                {{ months[date.getMonth()].toUpperCase() }}
              </th>
              <th colspan="1" @click="dateChange(0, 1)">
                <i class="bi bi-caret-right-fill h6 text-gray-300"></i>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <th scope="col" class="text-center text-blue-700">MON</th>
              <th scope="col" class="text-center text-blue-700">TUE</th>
              <th scope="col" class="text-center text-blue-700">WED</th>
              <th scope="col" class="text-center text-blue-700">THU</th>
              <th scope="col" class="text-center text-blue-700">FRI</th>
              <th scope="col" class="text-center text-blue-700">SAT</th>
              <th scope="col" class="text-center text-blue-700">SUN</th>
            </tr>
            <tr v-for="(_, index) in [0, 1, 2, 3, 4, 5]" :key="index">
              <td
                v-for="(_2, idx) in monthArray[index]"
                :key="idx"
                class="text-center border-1"
                :class="[
                  isSelectedDate(date, monthArray[index][idx]),
                  isCurrentDate(monthArray[index][idx]),
                  isNotCurrentMonth(date, monthArray[index][idx]),
                ]"
                v-hover="['bg-blue-700', 'text-gray-300']"
                @click="dateChosen(index, idx)"
              >
                {{ monthArray[index][idx].getDate() }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, toRefs, ref, computed } from "vue";

export default defineComponent({
  name: "DatePicker",
  props: {
    date: {
      required: true,
      type: Date,
    },
  },
  emits: ["selected-date"],
  directives: {
    hover: {
      beforeMount(el, binding) {
        const { value = [] } = binding;
        el.addEventListener("mouseenter", () => {
          el.classList.add(...value);
        });
        el.addEventListener("mouseleave", () => {
          el.classList.remove(...value);
        });
      },
      unmounted(el, binding) {
        el.removeEventListener("mouseenter", binding);
        el.removeEventListener("mouseleave", binding);
      },
    },
  },
  setup(props, context) {
    const { date } = toRefs(props);

    // Y, M, & D FOR DROPDOWN DISPLAY
    const dateYear = computed(() => {
      return date.value.getFullYear();
    });
    const dateMonthInt = computed(() => {
      return date.value.getMonth();
    });
    const dateMonthStr = computed(() => {
      let month = months[dateMonthInt.value];
      return month.length === 4
        ? month
        : months[dateMonthInt.value].slice(0, 3);
    });
    const dateDate = computed(() => {
      return ((date.value.getDate() < 10 ? "0" : "") +
        date.value.getDate()) as unknown as number;
    });

    // MONTH AND DAY LISTS
    const months = [
      "January",
      "February",
      "March",
      "April",
      "May",
      "June",
      "July",
      "August",
      "September",
      "October",
      "November",
      "December",
    ];
    const days = ["Sun", "Mon", "Tue", "Wed", "Thur", "Fri", "Sat"];

    // TOGGLE / SHOW
    const showDate = ref(false);
    const toggleDateView = () => {
      showDate.value = !showDate.value;
    };

    const dateChosen = (row: number, col: number) => {
      console.log(date.value, monthArray.value[row][col]);
      let newdate = monthArray.value[row][col];
      context.emit("selected-date", newdate);
    };
    /**********************************************************/
    // CALENDAR LOGIC
    // add this to all new dates (milliseconds)
    const tzOffset = computed(() => {
      return date.value.getTimezoneOffset() * 1000;
    });

    const getMonthLength = computed(() => {
      return 32 - new Date(dateYear.value, dateMonthInt.value, 32).getDate();
    });

    const getMonthFirstDay = computed(() => {
      let weekStartsOnSun = new Date(
        dateYear.value,
        dateMonthInt.value
      ).getDay();
      return weekStartsOnSun === 0 ? 6 : weekStartsOnSun - 1;
    });

    const monthArray = computed(() => {
      let tempArray = [];
      let firstDay = new Date(dateYear.value, dateMonthInt.value);
      for (let i = 0; i < 42; i++) {
        tempArray.push(
          new Date(
            firstDay.getTime() +
              i * 86400000 +
              tzOffset.value -
              getMonthFirstDay.value * 86400000
          )
        );
      }
      return [
        tempArray.slice(0, 7),
        tempArray.slice(7, 14),
        tempArray.slice(14, 21),
        tempArray.slice(21, 28),
        tempArray.slice(28, 35),
        tempArray.slice(35),
      ];
      // return monthArray;
    });
    // YEAR/MONTH ADVANCE
    const dateChange = (year: number, month: number) => {
      let tempdate;
      let newMonthLength =
        32 - new Date(dateYear.value, dateMonthInt.value + month, 32).getDate();
      if (year === 0 && dateDate.value > newMonthLength) {
        console.log(newMonthLength);
        tempdate = new Date(
          new Date(
            dateYear.value,
            dateMonthInt.value + month,
            newMonthLength
          ).getTime() + tzOffset.value
        );
      } else {
        tempdate = new Date(
          new Date(
            dateYear.value + year,
            dateMonthInt.value + month,
            dateDate.value
          ).getTime() + tzOffset.value
        );
      }

      context.emit("selected-date", tempdate);
    };

    // HIGHLIGHT SELECTED DATE
    const isSelectedDate = (date1: Date, date2: Date) => {
      return date1.getFullYear() === date2.getFullYear() &&
        date1.getMonth() === date2.getMonth() &&
        date1.getDate() === date2.getDate()
        ? "bg-red-500 text-bold text-gray-200"
        : "";
    };

    const isCurrentDate = (date: Date) => {
      let today = new Date(Date.now());
      return date.getFullYear() === today.getFullYear() &&
        date.getMonth() === today.getMonth() &&
        date.getDate() === today.getDate()
        ? "bg-blue-500 text-bold text-gray-200"
        : "";
    };

    const isNotCurrentMonth = (date1: Date, date2: Date) => {
      return date1.getFullYear() === date2.getFullYear() &&
        date1.getMonth() === date2.getMonth()
        ? "text-bold"
        : "text-gray-500";
    };

    return {
      dateYear,
      dateMonthStr,
      dateDate,
      showDate,
      toggleDateView,
      months,
      days,
      dateChosen,
      getMonthLength,
      getMonthFirstDay,
      tzOffset,
      monthArray,
      dateChange,
      isSelectedDate,
      isCurrentDate,
      isNotCurrentMonth,
    };
  },
});
</script>

<style lang="scss" scoped>
.cal-main-box {
  transform: translateX(-25%);
  height: 333px;
  width: 350px;
  border: solid;
  z-index: 100000;
}

.cal-small {
  width: 150px;
}

</style>
