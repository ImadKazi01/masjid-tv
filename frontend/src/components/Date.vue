<template>
  <div class="date-container">
    <h2>{{ date }}</h2>
    <h2>{{ hijri }}</h2>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";

const date = ref("");
const hijri = ref("");

onMounted(async () => {
  try {
    const response = await fetch(
      "https://api.sheety.co/aa3aaee63f4b8cdf31cb0f9e76e3b468/timetable/today"
    );
    if (!response.ok) throw new Error("Failed to fetch data");

    const { today } = await response.json();

    // Assuming the first two entries are for date and Hijri date
    date.value = `${today[0].name}`;
    hijri.value = `${today[1].name}`;
  } catch (error) {
    console.error("Error fetching timetable:", error);
  }
});
</script>

<style lang="scss" scoped>
.date-container {
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  margin: 0 2rem;

  h2 {
    text-align: center;
    color: #333;
    margin: 0px;
    padding: 0;
  }
}
</style>
