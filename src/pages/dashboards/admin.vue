<script setup lang="ts">

//set layout as none

const userDefaultApp = localStorage.getItem('userDefaultApp') as string
const userToken = localStorage.getItem('accessToken') as string

</script>

<template>
  <VRow class="match-height">
    <iframe allow="camera" :src="userDefaultApp + '?token=' + userToken"></iframe>
  </VRow>
</template>

<style lang="scss">
@use "@core/scss/template/libs/apex-chart.scss";
iframe{
  width: 100%;
  height: 100vh;
  border: none;
}
</style>
