<template>
  <div class="table-slider">
    <div v-if="response && response.iterations">
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
              <th
                v-for="(val, key) in iteration.table[0]"
                v-if="!['Ci', 'Vb', 'Bi', 'Øi'].includes(key)"
                :key="key"
                :class="{ 'pivot-cell': key === iteration.pivot_column }"
              >
                {{ key }}
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, index) in iteration.table" :key="index">
              <td>{{ row.Ci }}</td>
              <td>{{ row.Vb }}</td>
              <td>{{ row.Bi }}</td>
              <td>{{ row["Øi"] }}</td>
              <td
                v-for="(val, key) in row"
                v-if="!['Ci', 'Vb', 'Bi', 'Øi'].includes(key)"
                :key="key"
                :class="{ 'pivot-cell': key === iteration.pivot_column }"
              >
                {{ val }}
              </td>
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
                  <th v-for="(val, key) in iteration.cj_minus_zj" :key="key">
                    {{ key }}
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr>
                  <td v-for="(val, key) in iteration.cj_minus_zj" :key="key">
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
        <p>Valor Óptimo: {{ response.result.optimal_value }}</p>
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
</template>

<script>
export default {
  name: "SimplexTable",
  props: {
    response: {
      type: Object,
      required: true,
    },
  },
};
</script>

<style scoped>
.table-slider {
  display: flex;
  overflow-y: auto;
  gap: 20px;
  padding: 10px;
  width: 900px;
  max-height: 600px;
  scroll-snap-type: x mandatory;
}

.pivot-cell {
  border: 2px solid #ff5722;
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
  width: 100%;
  min-width: 600px;
  background-color: white;
  margin-bottom: 1rem;
}

th,
td {
  border: 1px solid #f4f4f4;
  padding: 15px;
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
