<template>
  <form @submit.prevent="onSubmit" class="form">
    <div class="form-variables">
      <label for="select-restrictions">Tipo de restricción</label>
      <select id="select-restrictions" v-model="restrictionType">
        <option value="max">>= (max)</option>
        <option value="min"><= (min)</option>
      </select>
    </div>

    <div class="form-variables">
      <label for="variables">Cantidad de variables</label>
      <input
        id="variables"
        type="number"
        min="1"
        max="5"
        placeholder="Max.5"
        v-model.number="variablesLength"
      />
    </div>

    <div class="form-restrictions">
      <label for="restrictions">Cantidad de restricciones</label>
      <input
        id="restrictions"
        type="number"
        min="1"
        max="5"
        placeholder="Max.5"
        v-model.number="restrictionsLength"
      />
    </div>

    <div class="form-actions">
      <button type="submit">Generar modelo</button>
      <button type="button" @click="resetForm">Limpiar</button>
    </div>
  </form>
</template>
<style scoped>
form {
  width: 500px;
  height: auto;
  min-height: 500px;
  max-height: max-content;
  background-color: rgb(255, 255, 255);
  border: 1px solid rgb(228, 228, 228);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-evenly;
  gap: 10px;
  position: relative;
}

form .form-variables,
.form-restrictions {
  width: 90%;
  max-height: max-content;
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

form .form-actions {
  width: 90%;
  height: 60px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

form .form-actions button {
  padding: 15px;
  outline: none;
  border: 1px solid rgb(182, 182, 182);
  border-radius: 5px;
  cursor: pointer;
  font-family: "Montserrat";
}

form .form-actions button:first-of-type {
  background-color: rgb(61, 61, 146);
  color: white;
}

form .form-actions button:last-of-type {
  background-color: transparent;
  border: none;
}

form input,
select {
  width: 100%;
  padding: 20px;
  box-sizing: border-box;
  border-radius: 5px;
  outline: none;
  border: 1px solid rgb(182, 182, 182);
}

form input::placeholder {
  font-family: "Montserrat", system-ui, Avenir, Helvetica, Arial, sans-serif;
}
</style>
<script setup>
import { ref } from "vue";

const restrictionType = ref("max");
const variablesLength = ref(0);
const restrictionsLength = ref(0);

const emit = defineEmits(["generateModel"]);

function onSubmit() {
  emit("generateModel", {
    restrictionType: restrictionType.value,
    variablesLength: variablesLength.value,
    restrictionsLength: restrictionsLength.value,
  });
}

function resetForm() {
  restrictionType.value = "max";
  variablesLength.value = 0;
  restrictionsLength.value = 0;
}
</script>
