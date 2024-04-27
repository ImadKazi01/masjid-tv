<script setup>
import { onMounted, ref } from "vue";

const times = ref([]);

async function fetchTimetable() {
  try {
    const response = await fetch(
      "https://script.googleusercontent.com/macros/echo?user_content_key=ASXOPfxON7PICP8dswC7DKS49Y1czjbThdAab31d9fL85ykT0L9Qwg6W4bpeV282vQ89RRHGNuohWMqUlVmVEuFhJQoQi2wyOJmA1Yb3SEsKFZqtv3DaNYcMrmhZHmUMWojr9NvTBuBLhyHCd5hHa2cy1fYhw-ptjotJHFe5-UkYfrZ-SsGsFINK7iIImb1jWNVU_IGfXKCcCZLSfRYC6uJ7_E8QeJqehtFGCy31_l7nl-_E56ncaRRXZqHQgsmyMXKfnN6wx2RSXWY044JlGpjz6loM1y2-&lib=MpdsG6ZlKXvlBlWo5rPhpDmE6ei3rOiSo"
    );
    if (!response.ok) throw new Error("Failed to fetch data");
    const data = await response.json();
    times.value = data.data.slice(3).map((entry) => ({
      name: entry.Name,
      startTime: entry["Start Time"] ? formatTime(entry["Start Time"]) : "",
      jamatTime: entry["Jamat Time"] ? formatTime(entry["Jamat Time"]) : "",
    }));
    console.log("Data fetched at:", new Date().toLocaleTimeString());
  } catch (error) {
    console.error("Error fetching timetable:", error);
  }
}

function formatTime(dateTimeString) {
  const timePart = dateTimeString.match(/T(\d{2}):(\d{2}):(\d{2})/);
  if (!timePart) return ""; // Return empty if no match found

  let hours = parseInt(timePart[1], 10);
  const minutes = timePart[2];

  hours = hours % 12;
  hours = hours || 12;

  return `${hours.toString().padStart(2, "0")}:${minutes}`;
}

function msUntilNextTarget(targetHour) {
  const now = new Date();
  const target = new Date(
    now.getFullYear(),
    now.getMonth(),
    now.getDate(),
    targetHour,
    0,
    0
  );
  if (now > target) {
    target.setDate(target.getDate() + 1);
  }
  return target - now;
}

// Set timeouts to fetch the data at specified hours
function scheduleFetches(fetchHours) {
  fetchHours.forEach((hour) => {
    const msUntilFetch = msUntilNextTarget(hour);
    setTimeout(() => {
      fetchTimetable();
      // Set interval to repeat every 24 hours
      setInterval(fetchTimetable, 24 * 60 * 60 * 1000);
    }, msUntilFetch);
  });
}

onMounted(() => {
  // Define the hours at which you want to fetch the data
  const fetchHours = [3, 9, 13, 17, 21]; // 24-hour format
  fetchTimetable(); // Fetch immediately on mount
  scheduleFetches(fetchHours);
});
</script>

<template>
  <div class="timetable-container">
    <table v-if="times.length">
      <thead>
        <tr>
          <th></th>
          <th>Start Time</th>
          <th>Jamat Time</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="time in times" :key="time.id">
          <td class="name">{{ time.name }}</td>
          <td class="start" :class="time.name === 'Jummah' ? 'jamat' : ''">
            {{ time.startTime }}
          </td>
          <td class="jamat">
            {{ time.jamatTime || "" }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style lang="scss" scoped>
.timetable-container {
  display: flex;
  width: 60%;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;

  table {
    width: 100%;
    height: 70vh;
    border-collapse: collapse;
    background: #2d9159;
    color: white;

    thead {
      tr {
        th {
          font-size: 1.5rem;
          padding-top: 2rem;
        }
      }
    }

    tbody {
      tr {
        td {
          font-size: 2.4rem;
        }
      }
    }
  }
}

.name,
.jamat {
  font-weight: bold;
}
</style>
