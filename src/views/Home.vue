<template>
  <Form @generateModel="handleGenerateModel" />

  <ModalForm
    v-if="showModal"
    :restrictionType="restrictionType"
    :variablesLength="variablesLength"
    :restrictionsLength="restrictionsLength"
    title="Calcular función objetivo"
    @close="showModal = false"
  />
</template>

<script setup>
import { ref } from "vue";
import Form from "../components/Form.vue";
import ModalForm from "../components/Modal-form.vue";

const restrictionType = ref(">=");
const variablesLength = ref(0);
const restrictionsLength = ref(0);
const showModal = ref(false);

function handleGenerateModel(payload) {
  restrictionType.value = payload.restrictionType;
  variablesLength.value = payload.variablesLength;
  restrictionsLength.value = payload.restrictionsLength;
  showModal.value = true;
}
</script>
