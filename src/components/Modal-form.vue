<template>
  <div class="overlay">
    <form @submit.prevent="handleSubmit">
      <button class="form-close" @click="closeModal">x</button>

      <h1>{{ title }}</h1>

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
            min="0"
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
              min="0"
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

      <div class="actions">
        <button type="submit">Enviar</button>
        <button type="button" @click="addRestriction">
          Agregar restricción
        </button>
      </div>

      <div v-if="response">
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
                      :class="{ 'pivot-cell': key === iteration.pivot_column }"
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
                      :class="{ 'pivot-cell': key === iteration.pivot_column }"
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
            <p :class="{'error' : response.result.error }">
              {{ response.result.optimal_value ? 'Valor Óptimo:' : ''}}
              {{ response.result.optimal_value || response.result.error }}
            </p>
            <table border="1">
              <thead>
                <tr>
                  <th v-for="(val, key) in response.result.solution" :key="key">
                    {{ key }}
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td v-for="(val, key) in response.result.solution" :key="key">
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
      restrictions: [],
      response: null,
    };
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
  },
  methods: {
    closeModal() {
      this.$emit("close");
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
    async handleSubmit() {
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
      } catch (err) {
        this.response = null;
        alert(`Error al enviar: ${err.message}`);
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

form {
  background: #fff;
  padding: 20px;
  border-radius: 10px;
  width: 50%;
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

.form-section {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.error{
  color: red;
}

.input-row {
  display: flex;
  gap: 10px;
  margin-block: 5px;
}

.form-section .restriction input {
  width: 80px;
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
  background-color: rgb(61, 61, 146);
  color: white;
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
}
</style>
