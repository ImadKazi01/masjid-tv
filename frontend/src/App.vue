<script setup>
import TimeTable from "./components/TimeTable.vue";
import Date from "./components/Date.vue";
</script>

<template>
  <div class="tv-display">
    <Date class="date-component" />
    <TimeTable class="timetable-component" />
  </div>
</template>

<style lang="scss">
.tv-display {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: flex-start;

  .date-component {
    margin-left: auto;
    display: flex;
    flex-direction: column;
    align-items: flex-end;
  }

  .timetable-component {
   margin-right: auto;
    width: 100%;
  }
}
</style>
