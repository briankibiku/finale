<template>
  <div>
    <AppWrapper requiresSmallAndLargeScreenViews="false">
      <template v-slot:large-screen> <Answers /></template
    ></AppWrapper>
  </div>
</template>

<script>
import AppWrapper from "../../components/AppWrapper.vue";
import Answers from "../../components/Answers.vue";
export default {
  loading: {
    color: "blue",
    height: "5px",
  },
  components: { AppWrapper, Answers },
};
</script>
