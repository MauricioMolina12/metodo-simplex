<template>
  <div class="overlay">
    <form @submit.prevent="handleSubmit">
      <button class="form-close" @click="closeModal">x</button>
      <div v-if="modalError" class="error-modal">
        <div class="error-modal-content">
          <p>{{ errorMessage }}</p>
          <button @click="modalError = false">Cerrar</button>
        </div>
      </div>

      <h1>{{ title }}</h1>

      <!-- <div class="edit-type-function">
        <label>Editar tipo de función</label>
        <select v-model="localRestrictionType">
          <option value="max">Max</option>
          <option value="min">Min</option>
        </select>
      </div> -->

      <div class="form-section">
        <label>Función Objetivo</label>
        <div
          class="input-row"
          v-for="(value, index) in objetiveFunctionValues"
          :key="index"
        >
          <input
            type="number"
            :placeholder="`Coeficiente X${index + 1}`"
            v-model="objetiveFunctionValues[index]"
          />
        </div>
      </div>

      <!-- Restricciones -->
      <div class="form-section">
        <label>Restricciones</label>
        <div
          class="restriction"
          v-for="(restriction, rIndex) in restrictions"
          :key="rIndex"
        >
          <div
            class="input-row"
            v-for="(val, vIndex) in restriction.values"
            :key="vIndex"
          >
            <input
              type="number"
              :placeholder="`X${vIndex + 1}`"
              v-model="restrictions[rIndex].values[vIndex]"
            />
          </div>

          <select v-model="restriction.sign">
            <option value="<=">≤</option>
            <option value=">=">≥</option>
            <option value="=">=</option>
          </select>

          <input
            type="number"
            placeholder="Valor"
            v-model="restriction.value"
          />
        </div>
      </div>

      <div v-if="objetiveFunctionValues.length" class="objective-display">
        <h4>Función Objetivo:</h4>
        <p>
          Z <span>({{ localRestrictionType }})</span> =
          <span v-for="(coef, index) in objetiveFunctionValues" :key="index">
            {{ index === 0 ? "" : coef >= 0 ? " + " : " - " }}
            {{ Math.abs(coef) }}x{{ index + 1 }}
          </span>
        </p>
      </div>

      <div v-if="restrictions.length" class="restrictions-preview">
        <h4>Vista previa de restricciones</h4>
        <ul>
          <li v-for="(restriction, rIndex) in restrictions" :key="rIndex">
            <span v-for="(val, vIndex) in restriction.values" :key="vIndex">
              <span v-if="vIndex > 0"> + </span>
              {{ val }}X{{ vIndex + 1 }}
            </span>
            <span> {{ restriction.sign }} {{ restriction.value }}</span>
          </li>
        </ul>
      </div>

      <div class="actions">
        <button type="submit" :disabled="loading">
          <span v-if="!loading">Enviar</span>
          <div v-else class="spinner"></div>
        </button>

        <button type="button" @click="addRestriction">
          Agregar restricción
        </button>

        <button type="button" @click="addVariable">Agregar variable</button>
      </div>

      <div v-if="response" ref="responseBlock">
        <transition name="fade-slide">
          <div v-if="response.iterations" class="iterations">
            <div
              v-for="iteration in response.iterations"
              :key="iteration.iteration"
              class="iteration-block"
            >
              <h3>Iteración {{ iteration.iteration }}</h3>
              <p v-if="iteration.pivot_column">
                Columna pivote: <strong>{{ iteration.pivot_column }}</strong>
              </p>
              <p v-else>Iteración final</p>

              <table border="1" class="table-simplex">
                <thead>
                  <tr>
                    <th>Ci</th>
                    <th>Vb</th>
                    <th>Bi</th>
                    <th>Øi</th>
                    <template v-for="(val, key) in iteration.table[0]">
                      <th
                        v-if="!['Ci', 'Vb', 'Bi', 'Øi'].includes(key)"
                        :key="key"
                        :class="{
                          'pivot-cell': key === iteration.pivot_column,
                        }"
                      >
                        {{ key }}
                      </th>
                    </template>
                  </tr>
                </thead>

                <tbody>
                  <tr v-for="(row, index) in iteration.table" :key="index">
                    <td>{{ row.Ci }}</td>
                    <td>{{ row.Vb }}</td>
                    <td>{{ row.Bi }}</td>
                    <td>{{ row["Øi"] }}</td>
                    <template v-for="(val, key) in row">
                      <td
                        v-if="!['Ci', 'Vb', 'Bi', 'Øi'].includes(key)"
                        :key="key"
                        :class="{
                          'pivot-cell': key === iteration.pivot_column,
                        }"
                      >
                        {{ val }}
                      </td>
                    </template>
                  </tr>
                </tbody>
              </table>

              <div class="summary-tables">
                <div>
                  <h4>Zj</h4>
                  <table border="1">
                    <thead>
                      <tr>
                        <th v-for="(val, key) in iteration.zj" :key="key">
                          {{ key }}
                        </th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr>
                        <td v-for="(val, key) in iteration.zj" :key="key">
                          {{ val }}
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </div>

                <div>
                  <h4>Cj - Zj</h4>
                  <table border="1">
                    <thead>
                      <tr>
                        <th
                          v-for="(val, key) in iteration.cj_minus_zj"
                          :key="key"
                        >
                          {{ key }}
                        </th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr>
                        <td
                          v-for="(val, key) in iteration.cj_minus_zj"
                          :key="key"
                        >
                          {{ val }}
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>

              <hr />
            </div>

            <div class="final-result">
              <h3>Resultado Final</h3>
              <p
                class="final-result"
                :class="{
                  success: response.result.optimal_value,
                  warning: response.result.error,
                }"
              >
                {{ response.result.optimal_value ? "Valor Óptimo:" : "" }}
                {{ response.result.optimal_value || response.result.error }}
              </p>
              <table border="1">
                <thead>
                  <tr>
                    <th
                      v-for="(val, key) in response.result.solution"
                      :key="key"
                    >
                      {{ key }}
                    </th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td
                      v-for="(val, key) in response.result.solution"
                      :key="key"
                    >
                      {{ val }}
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

          <div v-else>
            <p>No hay datos para mostrar.</p>
          </div>
        </transition>
      </div>

      <div class="history-toggle">
        <button @click="clearSavedExercises" v-if="savedExercises.length">
          Borrar historial
        </button>
      </div>

      <div class="saved-exercises" v-if="savedExercises.length">
        <h2>Ejercicios Guardados</h2>
        <ul>
          <li
            v-for="(exercise, index) in savedExercises"
            :key="index"
            class="exercise-card"
          >
            <h3>Ejercicio #{{ index + 1 }}</h3>
            <p class="objective-preview">
              <strong>Función Objetivo:</strong>
              Z ({{ exercise.optimization }}) =
              <span
                v-for="(val, i) in Object.values(exercise.objective)"
                :key="i"
              >
                {{ i !== 0 && val >= 0 ? " + " : i !== 0 ? " - " : "" }}
                {{ Math.abs(val) }}x{{ i + 1 }}
              </span>
            </p>
            <!-- <button @click="loadSavedExercise(exercise, index)">Ver</button> -->
          </li>
        </ul>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  emits: ["close"],
  props: {
    title: String,
    variablesLength: Number,
    restrictionType: String,
    restrictionsLength: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      objetiveFunctionValues: [],
      emptyFunctionValues: [],
      restrictions: [],
      emptyRestrictions: [],
      response: null,
      loading: false,
      modalError: false,
      errorMessage: "",
      localRestrictionType: this.restrictionType,
      savedExercises: [],
      activeExerciseIndex: null,
    };
  },
  mounted() {
    this.loadExercisesFromStorage();
  },
  watch: {
    variablesLength: {
      immediate: true,
      handler(newLength) {
        this.objetiveFunctionValues = Array.from(
          { length: newLength },
          () => ""
        );
        this.initRestrictions();
      },
    },
    restrictionsLength: {
      immediate: true,
      handler(newLength) {
        this.initRestrictions();
      },
    },

    localRestrictionType(newValue) {
      this.$emit("update:restrictionType", newValue);
    },
  },
  methods: {
    closeModal() {
      this.$emit("close");
    },
    loadExercisesFromStorage() {
      const raw = localStorage.getItem("simplex_exercises");
      this.savedExercises = raw ? JSON.parse(raw) : [];
    },

    saveExerciseToStorage(payload) {
      const current = JSON.parse(
        localStorage.getItem("simplex_exercises") || "[]"
      );

      if (this.activeExerciseIndex !== null) {
        current[this.activeExerciseIndex] = payload;
      } else {
        current.push(payload);
        this.activeExerciseIndex = current.length - 1;
      }

      localStorage.setItem("simplex_exercises", JSON.stringify(current));
      this.loadExercisesFromStorage();
    },

    loadSavedExercise(exercise, index) {
      this.activeExerciseIndex = index;

      this.objetiveFunctionValues = Object.values(exercise.objective).map(
        Number
      );
      this.localRestrictionType = exercise.optimization;
      this.restrictions = exercise.restrictions.map((res) => ({
        values: Object.keys(res)
          .filter((k) => k.startsWith("X"))
          .map((k) => Number(res[k])),
        sign: res.sign,
        value: Number(res.value),
      }));
      this.response = null;

      this.$nextTick(() => {
        const el = this.$refs.responseBlock;
        if (el?.scrollIntoView) {
          el.scrollIntoView({ behavior: "smooth" });
        }
      });
    },

    deleteSavedExercise(index) {
      this.savedExercises.splice(index, 1);
      localStorage.setItem(
        "simplex_exercises",
        JSON.stringify(this.savedExercises)
      );
    },

    clearSavedExercises() {
      localStorage.removeItem("simplex_exercises");
      this.savedExercises = [];
    },

    initRestrictions() {
      const nRes = this.restrictionsLength || 0;
      const nVar = this.variablesLength || 0;

      this.restrictions = [];
      for (let i = 0; i < nRes; i++) {
        this.restrictions.push({
          values: Array.from({ length: nVar }, () => ""),
          sign: ">=",
          value: "",
        });
      }
    },
    addRestriction() {
      this.restrictions.push({
        values: Array.from({ length: this.variablesLength }, () => ""),
        sign: ">=",
        value: "",
      });
    },

    addVariable() {
      this.objetiveFunctionValues.push("");
    },

    isEmpty(list, type) {
      if (type === "function") {
        return list
          .map((value, index) => (value === "" ? index : -1))
          .filter((i) => i !== -1);
      } else if (type === "restrictions") {
        return list
          .map((item, i) => {
            const emptyIndexes = item.values
              .map((val, j) => (val === "" ? j : -1))
              .filter((j) => j !== -1);
            return emptyIndexes.length > 0
              ? { restriction: i, positions: emptyIndexes }
              : null;
          })
          .filter((item) => item !== null);
      }
    },

    async handleSubmit() {
      this.loading = true;

      this.emptyFunctionValues = this.isEmpty(
        this.objetiveFunctionValues,
        "function"
      );
      this.emptyRestrictions = this.isEmpty(this.restrictions, "restrictions");

      if (
        this.emptyFunctionValues.length > 0 &&
        this.emptyRestrictions.length > 0
      ) {
        this.errorMessage =
          "Faltan datos en la función objetivo y en las restricciones.";
      } else if (this.emptyFunctionValues.length > 0) {
        const count = this.emptyFunctionValues.length;
        this.errorMessage =
          count === 1
            ? "Falta un coeficiente por escribir en la función objetivo."
            : `Faltan ${count} coeficientes por escribir en la función objetivo.`;
      } else if (this.emptyRestrictions.length > 0) {
        const count = this.emptyRestrictions.length;
        this.errorMessage =
          count === 1
            ? "Falta una restricción por completar."
            : `Faltan ${count} restricciones por completar.`;
      }

      if (this.errorMessage) {
        this.modalError = true;
        this.loading = false;
        return;
      }

      const objective = {};
      this.objetiveFunctionValues.forEach((val, i) => {
        objective[`X${i + 1}`] = val.toString();
      });

      const formattedRestrictions = this.restrictions.map((res) => {
        const result = {};
        res.values.forEach((val, i) => {
          result[`X${i + 1}`] = val.toString();
        });
        result.sign = res.sign;
        result.value = res.value.toString();
        return result;
      });

      const payload = {
        optimization: this.restrictionType,
        objective,
        restrictions: formattedRestrictions,
        M: 1000,
      };

      try {
        const res = await fetch(
          "https://simplex-production.up.railway.app/simplex",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(payload),
          }
        );

        if (!res.ok) throw new Error(`Error: ${res.status}`);
        const data = await res.json();
        this.response = data;

        this.saveExerciseToStorage({
          objective: this.objetiveFunctionValues,
          restrictions: this.restrictions,
          optimization: this.localRestrictionType,
        });

        this.$nextTick(() => {
          const el = this.$refs.responseBlock;
          if (el && el.scrollIntoView) {
            el.scrollIntoView({ behavior: "smooth" });
          }
        });
      } catch (err) {
        this.response = null;
        alert(`Error al enviar: ${err.message}`);
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
.overlay {
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
}

label {
  font-weight: 600;
  color: #4b8ac6;
}

form {
  background: #fff;
  padding: 20px;
  border-radius: 10px;
  width: 90%;
  max-height: 90%;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 20px;
  position: relative;
}

form .form-close {
  position: absolute;
  right: 4px;
  top: 4px;
  color: red;
  background-color: transparent;
  border: none;
  outline: none;
  font-size: 1.3rem;
  cursor: pointer;
}

.form-section,
.edit-type-function {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.form-section input,
.edit-type-function select {
  padding: 15px;
}

.edit-type-function select {
  width: 10%;
}

.error {
  color: red;
}

.input-row {
  display: flex;
  gap: 10px;
  margin-block: 5px;
}

.form-section .restriction input,
.form-section .restriction select {
  width: 80px;
  padding: 15px;
}

.restriction {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
}

.actions {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: start;
  gap: 10px;
  box-sizing: border-box;
  background-color: rgb(252, 252, 252);
  padding-bottom: 20px;
  border-bottom: 1px solid gray;
}

.actions button {
  padding: 15px;
  border-radius: 5px;
  outline: none;
  cursor: pointer;
  background-color: transparent;
  border: none;
  font-family: "Montserrat";
}

.actions button:first-of-type {
  background-color: rgb(0, 0, 0);
  color: white;
}

.actions button:nth-of-type(2) {
  background-color: #4b8ac6;
  color: white;
}

.actions button:last-of-type {
  border: 1px solid #4b8ac6;
}

input,
select {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

pre {
  background-color: #f5f5f5;
  padding: 10px;
  overflow-x: auto;
}

/* TABLE */

.pivot-cell {
  background-color: #fff3e0;
  font-weight: bold;
}

.iteration-table {
  flex: 0 0 auto;
  scroll-snap-align: start;
  min-width: 600px;
}

table {
  border-collapse: collapse;
  width: 80%;
  min-width: 600px;
  background-color: white;
  margin-bottom: 1rem;
  box-sizing: border-box;
}

th,
td {
  border: 1px solid #f4f4f4;
  padding: 8px;
  text-align: center;
  font-size: 0.95rem;
}

th {
  background-color: #d1e9ff;
  font-weight: bold;
  color: #333;
}

.iteration-block {
  margin-bottom: 2rem;
}
.table-simplex {
  width: 100%;
  margin-bottom: 1rem;
}
.summary-tables {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}
.summary-tables table {
  min-width: 300px;
}

.final-result {
  margin-top: 2rem;
  border-radius: 10px;
  padding: 10px;
  width: max-content;
}

.final-result.success {
  background-color: #e0ffe0;
  border: 1px solid green;
}

.final-result.success table thead tr th {
  background-color: #96ff96;
  border: 1px solid green;
}

.final-result.success h3 + p {
  font-weight: bold;
  color: green;
}

.final-result.warning {
  background-color: #ffcdcd;
  border: 1px solid #ff0707;
}

.final-result.warning table thead tr th {
  background-color: #ffcdcd;
  border: 1px solid #ff0707;
}

.final-result.warning h3 + p {
  font-weight: bold;
  color: #ff0707;
}

.spinner {
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-top: 3px solid white;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
  margin: auto;
}

.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.5s ease;
}
.fade-slide-enter-from,
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.iteration-block,
.summary-tables,
.final-result {
  animation: fadeIn 0.6s ease-in-out both;
}

.error-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.error-modal-content {
  background: white;
  padding: 20px 30px;
  border-radius: 8px;
  box-shadow: 0 0 10px #00000055;
  text-align: center;
}

.error-modal-content button {
  margin-top: 10px;
  padding: 8px 20px;
  background-color: crimson;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.objective-display {
  margin-top: 1rem;
  background-color: #d1e9ff;
  padding: 10px;
  border-radius: 5px;
}

.restrictions-preview {
  margin-top: 1rem;
  padding: 1rem;
  background-color: #eef2f7;
  border-radius: 6px;
}

.restrictions-preview ul {
  list-style-type: none;
  padding: 0;
}

.restrictions-preview li {
  margin-bottom: 0.5rem;
  margin-right: 20px;
}
.saved-exercises {
  margin-top: 2rem;
  padding: 1rem;
  background: #f5f7fa;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.saved-exercises h2 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: #333;
}

.saved-exercises ul {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  list-style: none;
  padding: 0;
  margin: 0;
}

.exercise-card {
  padding: 1rem;
  background: white;
  border-radius: 10px;
  border: 1px solid #e0e0e0;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.03);
  transition: transform 0.2s ease;
}

.exercise-card:hover {
  transform: scale(1.01);
}

.exercise-card h3 {
  margin: 0 0 0.5rem;
  font-weight: 600;
  font-size: 1.1rem;
  color: #444;
}

.objective-preview {
  margin: 0.5rem 0 1rem;
  font-size: 0.95rem;
  color: #555;
  font-family: monospace;
  background: #f0f3f7;
  padding: 0.5rem;
  border-radius: 6px;
}

.exercise-card button {
  padding: 0.4rem 0.8rem;
  background-color: #007acc;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background-color 0.2s ease;
}

.exercise-card button:hover {
  background-color: #005ea3;
}

.history-toggle {
  display: flex;
  gap: 1rem;
  margin-top: 1.5rem;
  flex-wrap: wrap;
}

.history-toggle button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 0.6rem 1rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.95rem;
  transition: background-color 0.3s ease;
}

.history-toggle button:last-of-type {
  background-color: #e53935;
}

.history-toggle button:hover {
  background-color: #45a049;
}

.history-toggle button:last-of-type:hover {
  background-color: #d2302e;
}

.clear-history {
  background-color: #e53935;
}

.clear-history:hover {
  background-color: #d32f2f;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(15px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
