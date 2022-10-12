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
  },
};
</script>

<template>
  <div class="flex justify-center bg-slate-600 h-screen p-2 overflow-auto">
    <div class="flex gap-2">
      <Datepicker
        v-model="date"
        :min-date="start"
        :max-date="end"
        :clearable="false"
        week-start="0"
        prevent-min-max-navigation
      ></Datepicker>
      <div class="w-64 md:w-96" dir="rtl">
        <div v-for="data in json" :key="data.id">
          <div
            v-if="
              checkProductMade(data.times.makeDays) &&
              checkAvailability(data.times.available_days_of_week) &&
              checkExclusion(data.times.excludeDates)
            "
            class="w-40 md:w-full rounded-md shadow-md bg-white p-4 mb-4"
          >
            <h1 class="font-bold text-lg">{{ data.name }}</h1>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
