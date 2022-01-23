<template>
  <div>
    <span class="text">
      {{ text }}
    </span>
    <!-- <span class="remove" @click="$emit('removeItem', text)">x</span> -->
  </div>
</template>

<script>
export default {
  name: "TestElement",
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
