<script setup>
import { onMounted, ref } from "vue";

const date = ref("");
const hijri = ref("");
const currentTime = ref("");

function updateTime() {
  const now = new Date();
  currentTime.value = now.toLocaleTimeString("en-GB", {
    hour: "numeric",
    minute: "2-digit",
    second: "2-digit",
    hour12: true,
  });
}

onMounted(async () => {
  try {
    const response = await fetch(
      "https://script.googleusercontent.com/macros/echo?user_content_key=ASXOPfxON7PICP8dswC7DKS49Y1czjbThdAab31d9fL85ykT0L9Qwg6W4bpeV282vQ89RRHGNuohWMqUlVmVEuFhJQoQi2wyOJmA1Yb3SEsKFZqtv3DaNYcMrmhZHmUMWojr9NvTBuBLhyHCd5hHa2cy1fYhw-ptjotJHFe5-UkYfrZ-SsGsFINK7iIImb1jWNVU_IGfXKCcCZLSfRYC6uJ7_E8QeJqehtFGCy31_l7nl-_E56ncaRRXZqHQgsmyMXKfnN6wx2RSXWY044JlGpjz6loM1y2-&lib=MpdsG6ZlKXvlBlWo5rPhpDmE6ei3rOiSo"
    );
    if (!response.ok) throw new Error("Failed to fetch data");

    const data = await response.json();
    const formatDate = new Date(data.data[0].Name);
    date.value = formatDate.toLocaleDateString("en-GB", {
      weekday: "long",
      day: "numeric",
      month: "long",
      year: "numeric",
    });
    hijri.value = `${data.data[1].Name}`;
  } catch (error) {
    console.error("Error fetching timetable:", error);
  }

  updateTime();
  setInterval(updateTime, 1000);
});
</script>

<template>
  <header>
    <div class="logo-container">
      <!-- Add your logo here -->
      <img
        src="https://lh4.googleusercontent.com/expOpBdj-uNqbHT9vvk05MCALOr3-w41VPx19m8LmlKSCMMXrTMMzugZz4JsTpfmYyXATA=w16383"
        class="lzy1Td"
        role="img"
        aria-label="Site home"
        jsname="SwcDWb"
      />
      <span>Madni Jamia Masjid</span>
    </div>
    <div class="date-time-container">
      <span class="date-time-container__time">{{ currentTime }}</span>
      <div class="date-time-container__date">
        <span>{{ date }}</span>
        <span>{{ hijri }}</span>
      </div>
    </div>
  </header>
</template>

<style lang="scss" scoped>
header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  height: 150px; /* Set the header height to 10% of the viewport height */
  width: 100%;
  background-color: #faf0e6;

  .date-time-container {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    width: 60%;
    text-align: center;
    gap: 1rem;
    height: 100%; /* Make the date-time container fill the header height */
    border-left: solid 1px black;

    span {
      display: block;
      color: black;
      margin: 0;
      padding: 0 2rem;
    }

    &__date {
      display: flex;
      flex-direction: column;
      font-size: 2.3rem;
      font-weight: 600;
      align-items: flex-end;
    }

    &__time {
      font-size: 4rem;
      font-weight: bold;
      text-transform: uppercase;
    }
  }

  .logo-container {
    display: flex;
    width: 40%;
    justify-content: center;
    align-items: center;
    gap: 2rem;
    height: 100%;

    img {
      width: 100%;
      max-width: 120px; 
      height: auto;
    }

    span {
      font-size: 2rem;
      font-weight: bold;
    }
  }
}
</style>
