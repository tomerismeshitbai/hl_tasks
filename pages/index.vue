<template>
  <title>Weather-app</title>
  <div class="min-h-screen bg-blue-100 py-8">
    <WeatherForm />
  </div>
</template>

<script>
import WeatherForm from '~/components/WeatherForm.vue';

export default {
  components: {
    WeatherForm,
  },
};
</script>
