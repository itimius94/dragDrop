<template>
  <div>
    <span class="text"> Nguyen Van Thuc </span>
  </div>
</template>
<style>
</style>
<script>
export default {
  name: "TestElement2",
  props: {
    text: {
      type: String,
      default: "x",
    },
  },
  data: function () {
    return {};
  },
  mounted: function () {
    console.log("### " + this.text + " ready!");
  },
};
</script>
