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
        "Asar today",
        "Maghrib today",
        "Isha today",
        "Jummah Khutbah",
      ].includes(entry.Name)
    );

    times.value = filteredData;
    console.log("Data fetched at:", new Date().toLocaleTimeString());

    // Schedule the next fetch after 5 minutes of each Jamat time
    const nextFetchTime = getNextFetchTime(filteredData, currentTime.value);
    const delay = nextFetchTime - currentTime.value;
    setTimeout(fetchTimetable, delay);
  } catch (error) {
    console.error("Error fetching timetable:", error);
  }
}

function getNextFetchTime(data, currentTime) {
  const jamatTimes = data
    .filter((entry) => entry["Jamat Time"])
    .map((entry) => new Date(entry["Jamat Time"]));

  const nextJamatTime = jamatTimes.find((time) => time > currentTime);

  if (nextJamatTime) {
    // Add 5 minutes to the next Jamat time
    return new Date(nextJamatTime.getTime() + 5 * 60000);
  } else {
    // If no more Jamat times today, fetch at midnight (start of the next day)
    const midnight = new Date(currentTime);
    midnight.setHours(24, 0, 0, 0);
    return midnight;
  }
}

const formattedTimes = computed(() => {
  return times.value.map((entry, index) => {
    const formattedStartTime = formatTime(entry["Start Time"]);
    const formattedJamatTime = formatTime(entry["Jamat Time"]);
    const isActive = isTimeActive(
      entry["Start Time"],
      times.value[index + 1]?.["Start Time"],
      currentTime.value
    );
    return {
      ...entry,
      "Start Time": formattedStartTime,
      "Jamat Time": formattedJamatTime,
      isActive,
    };
  });
});

function formatTime(timeString) {
  if (!timeString) return "";

  const [hours, minutes] = timeString.split("T")[1].split(":");
  const hour = parseInt(hours, 10);
  const formattedHour = hour % 12 === 0 ? 12 : hour % 12;
  return `${formattedHour}:${minutes}`;
}

function isTimeActive(startTime, nextStartTime, currentTime) {
  if (!startTime) return false;

  const startDateTime = new Date(startTime);
  const nextDateTime = nextStartTime
    ? new Date(nextStartTime)
    : new Date(startDateTime);
  nextDateTime.setDate(nextDateTime.getDate() + 1); // Set next day if no more prayers today

  return currentTime >= startDateTime && currentTime < nextDateTime;
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
          <td
            v-if="time['Start Time'] && !time['Jamat Time']"
            class="jamat"
            colspan="2"
          >
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
