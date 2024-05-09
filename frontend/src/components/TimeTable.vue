<script setup>
import { onMounted, ref, computed } from "vue";

const times = ref([]);
const currentTime = ref(new Date());

async function fetchTimetable() {
  try {
    const response = await fetch(import.meta.env.VITE_GOOGLE_TIMETABLE_API_URL);
    if (!response.ok) throw new Error("Failed to fetch data");
    const data = await response.json();

    const filteredData = data.data.filter((entry) =>
      [
        "Fajr tomorrow",
        "Sunrise tomorrow",
        "Zohar today",
        "Zohar tomorrow",
        "Asar today",
        "Maghrib today",
        "Isha today",
        "Jummah Khutbah",
      ].includes(entry.Name)
    );

    times.value = filteredData;
    console.log("Data fetched at:", new Date().toLocaleTimeString());

    // Schedule the next page reload based on Jamat times
    scheduleNextReload();
  } catch (error) {
    console.error("Error fetching timetable:", error);
  }
}

function scheduleNextReload() {
  const now = new Date();
  const jamatTimes = times.value
    .filter((entry) => entry["Jamat Time"])
    .map((entry) => new Date(entry["Jamat Time"]));

  let nextReloadTime = null;

  for (const jamatTime of jamatTimes) {
    const reloadTime = new Date(jamatTime);
    reloadTime.setMinutes(reloadTime.getMinutes() + 5);

    if (reloadTime > now) {
      nextReloadTime = reloadTime;
      break;
    }
  }

  if (nextReloadTime) {
    const delay = nextReloadTime - now;
    setTimeout(() => {
      location.reload();
    }, delay);
  }
}

const formattedTimes = computed(() => {
  return times.value.map((entry) => {
    const formattedStartTime = formatTime(entry["Start Time"]);
    const formattedJamatTime = formatTime(entry["Jamat Time"]);
    return {
      ...entry,
      "Start Time": formattedStartTime,
      "Jamat Time": formattedJamatTime,
    };
  });
});

console.log(formattedTimes);

function formatTime(timeString) {
  if (!timeString) return "";

  const [hours, minutes] = timeString.split("T")[1].split(":");
  const hour = parseInt(hours, 10);
  const formattedHour = hour % 12 === 0 ? 12 : hour % 12;
  return `${formattedHour}:${minutes}`;
}

onMounted(() => {
  fetchTimetable(); // Fetch immediately on mount
  setInterval(() => {
    currentTime.value = new Date(); // Update current time every second
  }, 1000);
});
</script>

<template>
  <div class="timetable-container">
    <table v-if="formattedTimes.length">
      <thead>
        <tr>
          <th></th>
          <th>Start Time</th>
          <th>Jamat Time</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="time in formattedTimes"
          :key="time.id"
          :class="{
            jummah: time.Name === 'Jummah Khutbah',
          }"
        >
          <td class="name">{{ time.Name }}</td>
          <td v-if="time['Start Time'] && !time['Jamat Time']" class="jamat" colspan="2">
            {{ time["Start Time"] }}
          </td>
          <td
            v-else-if="time['Start Time'] === time['Jamat Time']"
            class="jamat"
            colspan="2"
          >
            {{ time["Jamat Time"] }}
          </td>
          <td
            v-else-if="time['Start Time'] !== time['Jamat Time']"
            class="start"
            :class="time.Name === 'Jummah Khutbah' ? 'jummah' : ''"
          >
            {{ time["Start Time"] }}
          </td>
          <td
            v-if="time['Start Time'] !== time['Jamat Time']"
            class="jamat"
            :class="time.Name === 'Jummah Khutbah' ? 'jummah' : ''"
          >
            {{ time["Jamat Time"] }}
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style lang="scss" scoped>
.timetable-container {
  display: flex;
  width: 40%;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;

  table {
    width: 100%;
    height: auto;
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
          font-size: 2.7rem;
          text-align: center;
        }

        &.jummah {
          background-color: #ffad1f; /* Change the color as desired */
        }

        .jamat-only {
          text-align: center;
        }
      }
    }
  }
}

.name,
.jamat,
.jummah {
  font-weight: bold;
}
</style>
