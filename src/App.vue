<script lang="ts">
import json from "./assets/api.json";
import Datepicker from "@vuepic/vue-datepicker";
import "@vuepic/vue-datepicker/dist/main.css";

export default {
  components: { Datepicker },
  data() {
    return {
      json: json.data,
      start: new Date(),
      end: new Date(new Date().getTime() + 14 * 24 * 60 * 60 * 1000),
      date: new Date(),
      daysHebrew: ["ראשון", "שני", "שלישי", "רביעי", "חמישי", "שישי", "שבת"],
      recieveDay: [0, ""], // [day, date] tuple
    };
  },
  methods: {
    // make sure product is made at the requested day
    checkProductMade(makeDays: number) {
      return makeDays < this.date.getDay() + 1;
    },
    // make sure product is available on selected date's day
    checkAvailability(daysArray: number[]) {
      return daysArray.includes(this.date.getDay() + 1);
    },
    // make sure product is not excluded on selected date
    checkExclusion(
      dates: {
        date: string;
        id: string;
      }[]
    ) {
      return (
        dates.filter(
          (obj) =>
            new Date(obj.date).toString().split(" ").splice(0, 4).join(" ") ===
            this.date.toString().split(" ").splice(0, 4).join(" ")
        ).length === 0
      );
    },
    // function that returns which day the order can be recieved
    // based on the product's makeDays, availability and exclusions
    getRecieveDay(
      makeDays: number,
      daysArray: number[],
      dates: {
        date: string;
        id: string;
      }[]
    ): [number, string] {
      const date = new Date();
      date.setDate(this.date.getDate() + makeDays);
      while (
        !this.checkAvailability(daysArray) ||
        !this.checkExclusion(dates)
      ) {
        date.setDate(date.getDate() + 1);
      }
      return [date.getDay(), date.toLocaleDateString("en-il")];
    },
  },

  computed: {
    filteredProducts() {
      return this.json.filter((item) => {
        return (
          this.checkAvailability(item.times.available_days_of_week) &&
          this.checkExclusion(item.times.excludeDates)
        );
      });
    },
  },
};
</script>

<template>
  <div class="flex justify-center bg-slate-600 h-screen p-2 overflow-auto">
    <div>
      <div class="flex gap-2">
        <Datepicker
          v-model="date"
          :min-date="start"
          :max-date="end"
          :clearable="false"
          week-start="0"
          prevent-min-max-navigation
        ></Datepicker>
        <div class="w-64 md:w-96 grid grid-cols-2 gap-2" dir="rtl">
          <template v-for="data in filteredProducts" :key="data.id">
            <div
              class="w-24 md:w-full rounded-md shadow-md bg-white p-4 mb-4 h-28"
              :set="
                (recieveDay = getRecieveDay(
                  data.times.makeDays,
                  data.times.available_days_of_week,
                  data.times.excludeDates
                ))
              "
            >
              <h1 class="font-bold text-sm md:text-lg">{{ data.name }}</h1>
              <p class="text-xs md:text-sm">
                ניתן להזמין את המוצר ליום
                <!-- day in week -->
                {{ daysHebrew[recieveDay[0] as number] }}
                {{ " " }}
                <!-- date formatted as dd/mm/yyyy -->
                {{ recieveDay[1] }}
              </p>
            </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>
