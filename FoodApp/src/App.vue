<template>
  <div class="min-h-screen bg-pink-50 flex flex-col items-center justify-center gap-6 p-4">
    <div class="w-full max-w-4xl">
      <MealCreator />
      <div class="my-6"></div>
      <WeeklyPlanner />
    </div>
  </div>
</template>

<script>
import MealCreator from "./components/MealCreator.vue"
import WeeklyPlanner from "./components/WeeklyPlanner.vue"

export default {
  components: { MealCreator, WeeklyPlanner }
}
</script>
