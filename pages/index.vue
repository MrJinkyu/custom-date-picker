<template>
  <div>
    <CustomDatepicker v-model="reservationDate" :allow-manual-input="true"/>
    <!-- allow-manual-input props 기본값은 false -->
    <!-- allow-manual-input props가 true면 키보드로 날짜 입력 가능 -->
  </div>
</template>

<script setup lang="ts">
const reservationDate  = ref('');
</script>

<style scoped>

</style>