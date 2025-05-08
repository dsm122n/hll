<script setup>
import { ref } from 'vue'
import json from '../assets/lab.json'

const transformData = (data) => {
  const keys = Object.keys(data);
  const rowCount = data[keys[0]].length; 
  const rows = [];

  for (let i = 0; i < rowCount; i++) {
    const row = {};
    keys.forEach((key) => {
      row[key] = data[key][i];
    });
    rows.push(row);
  }

  return rows;
};

const lab = ref(transformData(json));
// Reactive states for highlighting
const highlightedRow = ref(null);
const highlightedColumn = ref(null);

// Highlight functions
const highlightRow = (rowIndex) => {
  highlightedRow.value = rowIndex;
};

const highlightColumn = (colIndex) => {
  highlightedColumn.value = colIndex;
};

const highlightCell = (rowIndex, colIndex) => {
  highlightedRow.value = rowIndex;
  highlightedColumn.value = colIndex;
};

const clearHighlight = () => {
  highlightedRow.value = null;
  highlightedColumn.value = null;
};

</script>

<template>
  <h1>Flujo de examenes</h1>

    <div class="table-container">

        <table class="vertical">
            <thead>
            <tr>
                <!-- Crear encabezados basados en las claves del primer elemento -->
                <th v-for="key in Object.keys(lab[0])" :key="key">{{ key }}</th>
            </tr>
            </thead>
            <tbody>
        <!-- Crear filas dinámicamente -->
        <tr
          v-for="(item, rowIndex) in lab"
          :key="rowIndex"
          @mouseover="highlightRow(rowIndex)"
          @mouseleave="clearHighlight"
          :class="{'highlight-row': highlightedRow === rowIndex}"
        >
          <td
            v-for="(value, colIndex) in Object.values(item)"
            :key="colIndex"
            :class="{
              'highlight-col': highlightedColumn === colIndex,
              'highlight-row': highlightedRow === rowIndex
            }"
            @mouseover="highlightCell(rowIndex, colIndex)"
            @mouseleave="clearHighlight"
          >
            {{ value }}
          </td>
        </tr>
      </tbody>
        </table>
    </div>
    
</template>

<style>
.vertical {
  display: -ms-grid;
  -ms-grid-rows: auto;
  -ms-grid-columns: auto auto;
}
.vertical thead {
  -ms-grid-row: 1;
  -ms-grid-column: 1;
}
.vertical tbody {
  -ms-grid-row: 1;
  -ms-grid-column: 2;
}

/* Everyone Else's Grid */
@supports (display: grid) {
  .vertical {
    display: grid;
    grid-template-columns: min-content min-content;
    grid-template-areas: "head body";
  }
  .vertical thead {
    grid-area: head;
  }
  .vertical tbody {
    grid-area: body;
  }
}

/* Flex - Cross Browser CSS */
.vertical thead {
  display: flex;
  flex-shrink: 0;
  min-width: max-content;
}
.vertical tbody {
  display: flex;
  min-width: max-content;

}
.vertical tr {
  display: flex;
  flex-direction: column;
  min-width: min-content;
  flex-shrink: 0;
}
.vertical td, .vertical th {
  display: block;
  padding: 0 1em 0 1em;
}

.table-container {
  overflow-x: auto;
  width: 100%;
}

table { 
    border-collapse: collapse; 
}
table td {
  border: 1px solid black; 
  background-color: #ffffff00;
}
table th {
  border: 1px solid black;
  background-color: #ffc800;
  color: rgb(0, 0, 0);
  text-align: left;
}
/* change background in groups of 5 */
table tr:nth-child(10n), tr:nth-child(10n-1), tr:nth-child(10n-2), tr:nth-child(10n-3), tr:nth-child(10n-4) {
  background-color: #d2d2d2;
  color: #000;
}
/* on hover, change background color of columns and rows */
.highlight-row {
  background-color: #ffa2e577; /* Light pink */
}

.highlight-col {
  background-color: #00ffbb33; /* Light purple */
}

@media (preffers-color-scheme: light) {
  .highlight-row {
    background-color: #ff77d877; /* Light pink */
  }

  .highlight-col {
    background-color: #1ff0b833; /* Light purple */
  }
}


</style>