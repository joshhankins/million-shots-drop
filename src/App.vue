<template>
  <div>
    <QuestionDisplay />
  </div>
</template>

<script>
import QuestionDisplay from "./components/QuestionDisplay.vue";

export default {
  components: {
    QuestionDisplay,
  },
};
</script>

<style>
/* Global styles if needed */
body {
  margin: 0;
  font-family: Arial, sans-serif;
  background-color: #121212;
  color: #fff;
}
</style>