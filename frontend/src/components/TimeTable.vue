<script setup>
import { onMounted, ref } from "vue";

const times = ref([]);

// Function to fetch the timetable data
async function fetchTimetable() {
  try {
    const response = await fetch(
      "https://api.sheety.co/aa3aaee63f4b8cdf31cb0f9e76e3b468/timetable/today"
    );
    if (!response.ok) throw new Error("Failed to fetch data");

    const { today } = await response.json();
    times.value = today.slice(2);
    console.log("Data fetched", new Date().toLocaleTimeString());
  } catch (error) {
    console.error("Error fetching timetable:", error);
  }
}

// Schedule the fetch operation to run at the next target time
function scheduleNextFetch(targetHours) {
  // Find the next target time
  const now = new Date();
  const today = now.toISOString().slice(0, 10);
  const targetTimes = targetHours.map((hour) =>
    new Date(`${today}T${hour.padStart(2, "0")}:00:00`).getTime()
  );
  const nextTime = targetTimes.find((time) => time > now.getTime());

  // Calculate delay until the next target time
  const delay = nextTime ? nextTime - now.getTime() : null;

  if (delay !== null) {
    setTimeout(() => {
      fetchTimetable();
      // After fetching, schedule the next fetch operation
      scheduleNextFetch(targetHours);
    }, delay);
  } else {
    // If today's target times are past, schedule the first fetch for the next day
    const tomorrowFirstTargetTime = new Date(
      targetTimes[0] + 24 * 60 * 60 * 1000
    );
    setTimeout(() => {
      fetchTimetable();
      // Schedule the next day's fetches
      scheduleNextFetch(targetHours);
    }, tomorrowFirstTargetTime.getTime() - now.getTime());
  }
}

onMounted(() => {
  // Define the hours at which you want to fetch the data
  const fetchHours = ["03:00", "13:00", "20:00"];
  scheduleNextFetch(fetchHours);
});
</script>

<template>
  <div class="timetable-container">
    <table v-if="times.length">
      <thead>
        <tr>
          <th>Salah</th>
          <th>Start Time</th>
          <th>Jamat Time</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="time in times" :key="time.id">
          <td class="name">{{ time.name }}</td>
          <td>{{ time.startTime }}</td>
          <td class="jamat">{{ time.jamatTime || "" }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style lang="scss" scoped>
.timetable-container {
  max-width: 50%;
  width: 100%;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;

  h2 {
    text-align: center;
    color: #333;
    margin-bottom: 20px;
  }

  table {
    width: 100%;
    height: 70vh;
    border-collapse: collapse;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

    thead {
      tr {
        background-color: #007bff;
        color: #ffffff;

        th {
          padding: 10px;
          border: 1px solid #007bff;
        }
      }
    }

    tbody {
      tr {
        &:nth-child(odd) {
          background-color: #f2f2f2;
          color: #333;
        }

        td {
          padding: 8px;
          border: 1px solid #ddd;
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
