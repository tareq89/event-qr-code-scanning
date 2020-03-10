<template>
  <section class="container">
    <b-tabs content-class="mt-3">
      <b-tab title="Attendance Check" active>
        <Attendance />
      </b-tab>
      <b-tab title="Food Check">
        <FoodCheck />
      </b-tab>
    </b-tabs>
  </section>
</template>

<script>
import Attendance from '~/components/Attendance.vue'
import FoodCheck from '~/components/FoodCheck.vue'

export default {
  components: {
    Attendance,
    FoodCheck
  }
}
</script>

<style>
.container
{
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
</style>
