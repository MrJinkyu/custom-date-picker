<template>
  <div>
    <CustomDatepicker v-model="reservationDate" :allow-manual-input="true" type="dateTime"/>
     <p>선택한 날짜: {{ reservationDate }}</p>
  </div>
</template>

<script setup lang="ts">
const reservationDate  = ref('');
</script>

<style scoped>

</style>