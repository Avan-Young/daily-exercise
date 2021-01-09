<template>
  <div id="app">
    <!-- 刚好这里可以用上双向绑定，就省了父组件的methods -->
    <!-- v-model默认绑定的:value @input 在子组件使用model选项修改为 :date @choose-date -->
    <!-- <date-picker :date="date" @choose-date="chooseDate" /> -->

    <date-picker v-model="date"></date-picker>
  </div>
</template>
<script>
import DatePicker from "./DatePicker";
export default {
  components: {
    DatePicker,
  },
  data() {
    return {
      date: new Date(),
    };
  },
  /* methods: {
    chooseDate(date) {
      this.date = date;
    },
  }, */
};
</script>

<style scoped>
</style>