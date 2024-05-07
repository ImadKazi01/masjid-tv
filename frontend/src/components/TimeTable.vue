<script setup>
import { onMounted, ref } from "vue";

const times = ref([]);

async function fetchTimetable() {
  try {
    const response = await fetch(
      "https://script.googleusercontent.com/macros/echo?user_content_key=EYt-iY7GV2zvrX9X07ELKluXbUkzx75DpUE_70bwtFwdxTXoRUOo_htxoUbA-NNqoDdZC9_Vvee1wfwTgI_4uNWb4_AMSJOzOJmA1Yb3SEsKFZqtv3DaNYcMrmhZHmUMWojr9NvTBuBLhyHCd5hHawDD7f-ZhjdUHo0eo4bAqwscOgQ_LEHiRqOxz3jNEaSzz5QBRU34F3h_6i07jeuS36ORojFJlsM9SAns65DBbiiGHg4Tv-3CrRRXZqHQgsmy0237Oe_IY2RSXWY044JlGpjz6loM1y2-&lib=MEDXf46DmJ1Z0zvQ5xfCtIWE6ei3rOiSo"
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
  const fetchHours = [3, 9, 14, 19, 21]; // 24-hour format
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
        <tr
          v-for="time in times"
          :key="time.id"
          :class="time.name === 'Jummah' ? 'jummah' : ''"
        >
          <td class="name">{{ time.name }}</td>
          <td
            v-if="time.startTime && !time.jamatTime"
            class="jamat"
            colspan="2"
          >
            {{ time.startTime }}
          </td>
          <td
            v-else-if="time.startTime === time.jamatTime"
            class="jamat"
            colspan="2"
          >
            {{ time.jamatTime }}
          </td>
          <td
            v-else-if="time.startTime !== time.jamatTime"
            class="start"
            :class="time.name === 'Jummah' ? 'jummah' : ''"
          >
            {{ time.startTime }}
          </td>
          <td
            v-if="time.startTime !== time.jamatTime"
            class="jamat"
            :class="time.name === 'Jummah' ? 'jummah' : ''"
          >
            {{ time.jamatTime }}
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
          font-size: 2.4rem;
        }
      }
    }
  }
}

.jummah {
  background: #e8a318;
}

.name,
.jamat,
.jummah {
  font-weight: bold;
}
</style>
