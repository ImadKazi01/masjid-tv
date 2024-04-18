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
      "https://script.googleusercontent.com/macros/echo?user_content_key=ASXOPfxON7PICP8dswC7DKS49Y1czjbThdAab31d9fL85ykT0L9Qwg6W4bpeV282vQ89RRHGNuohWMqUlVmVEuFhJQoQi2wyOJmA1Yb3SEsKFZqtv3DaNYcMrmhZHmUMWojr9NvTBuBLhyHCd5hHa2cy1fYhw-ptjotJHFe5-UkYfrZ-SsGsFINK7iIImb1jWNVU_IGfXKCcCZLSfRYC6uJ7_E8QeJqehtFGCy31_l7nl-_E56ncaRRXZqHQgsmyMXKfnN6wx2RSXWY044JlGpjz6loM1y2-&lib=MpdsG6ZlKXvlBlWo5rPhpDmE6ei3rOiSo"
    );
    if (!response.ok) throw new Error("Failed to fetch data");

    const data = await response.json();
    // Assuming the first two entries are for date and Hijri date
    const formatDate = new Date(data.data[0].Name);
    date.value = formatDate.toLocaleDateString("en-GB", {
      weekday: "long", // "Thursday"
      day: "numeric", // "18"
      month: "long", // "April"
      year: "numeric", // "2024"
    });
    hijri.value = `${data.data[1].Name}`;
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
