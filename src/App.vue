<script setup>
import { ref } from "vue";
import Form from "./components/Form.vue";
import ModalForm from "./components/Modal-form.vue";
import Table from "./components/Table.vue";
</script>
<template>
  <div class="layout">
    <main class="main-container">
      <h1>Método simplex calculator</h1>
      <h2>Por el momento no está adaptado en móviles. Usa una pc :></h2>
      <router-view />
    </main>
  </div>
</template>

<style>
body {
  margin: 0;
  font-family: "Montserrat";
}

.layout {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  box-sizing: border-box;
  display: flex;
  align-items: center;
}

.layout .main-container {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  box-sizing: border-box;
  background-color: rgb(238, 238, 238);
  border: 1px solid rdd;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.layout .main-container h1 {
  color: rgb(53, 53, 53);
}

.layout .sidebar {
  width: 10%;
  height: 100%;
  background-color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  border-right: 1px solid rgb(225, 225, 225);
}

.layout .sidebar .sidebar__header {
  width: 100%;
  height: 10%;
  display: flex;
  justify-content: end;
  align-items: center;
  gap: 10px;
  padding-right: 10px;
}

.layout .sidebar__header button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 9%;
  min-width: 30px;
  border-radius: 4px;
  outline: none;
  border: 1px solid rgb(225, 225, 225);
  cursor: pointer;
}

.sidebar__header span {
  color: gray;
}

.main-container h2 {
  display: none;
}

@media (max-width: 900px) {
  .main-container {
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .main-container h2 {
    display: block;
    text-align: center;
    font-size: 1.2rem;
    max-width: 80%;
  }
}
</style>
