<template>
  <Table :response="object" />
</template>
<script>
import Table from "../components/Table.vue";


export default {
  name: "Response",
  props: {
    Response: {
      type: Object,
      required: true,
    },
  },
};
</script>
<style></style>
