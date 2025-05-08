<script setup>
import { ref, computed, watch } from 'vue';
import { labResults } from "../stores/labResultsStore.js";
import { LabResultClass } from "../models/LabResultsClass.js";

const datitosLab = ref([]);

const highlightedRow = ref(null);
const highlightedColumn = ref(null);

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

// Define the table headers based on LabResultClass properties
const tableHeaders = Object.keys(new LabResultClass());

// Watch for changes in labResults and update datitosLab
watch(labResults, (newVal) => {
    datitosLab.value = newVal;
    console.log("Updated datitosLab:", datitosLab.value);
}, { immediate: true });

// Computed property to format the date
const formatDate = (date) => {
    if (!date) return ' ';
    const d = new Date(date);
    const day = String(d.getDate()).padStart(2, '0');
    const month = String(d.getMonth() + 1).padStart(2, '0');
    const year = d.getFullYear();
    return `${day}/${month}/${year}`;
};

// Function to delete a row
const deleteRow = (index) => {
    labResults.value.splice(index, 1);
};
</script>

<template>
  <h1>Flujo de Exámenes</h1>
  <div v-if="datitosLab.length > 0">
    <div class="table-container" id="table-container">
      <p class="only-when-print" style="margin:0; padding: 0.25em;">Rut:_________________ Nombre:_____________________________________________________________________________ Fecha:____/____/____</p>
      <table class="vertical" id="table-for-print">
        <thead>
          <tr>
            <th class="eliminar-columnas" >Eliminar --> </th>
            <th v-for="key in tableHeaders" :key="key"> {{ key }} </th>
            <th style="height: 1.5em;"></th>
            <th style="height: 1.5em;"></th>
            <th style="height: 1.5em;"></th>
            <th style="height: 1.5em;"></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(result, rowIndex) in datitosLab" :key="result.fileName"
              @mouseover="highlightRow(rowIndex)"
              @mouseleave="clearHighlight"
              :class="{ 'highlight-row': highlightedRow === rowIndex }">
              <td class="eliminar-columnas" >
                <button @click="deleteRow(rowIndex)" class="boton-eliminar-columnas"></button>
              </td>
            <td v-for="(key, colIndex) in tableHeaders" :key="colIndex"
                @mouseover="highlightCell(rowIndex, colIndex)"
                @mouseleave="clearHighlight"
                :class="{ 
                  'highlight-row': highlightedRow === rowIndex,
                  'highlight-col': highlightedColumn === colIndex
                }">
              {{ key === 'Fecha' ? formatDate(result.extractedData[key]) : (result.extractedData[key] || ' ') }}
            </td>
            <td style="height: 1.5em;"></td>
            <td style="height: 1.5em;"></td>
            <td style="height: 1.5em;"></td>
            <td style="height: 1.5em;"></td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
  <div v-else>
    <p>No hay resultados de laboratorio disponibles.</p>
  </div>
</template>

<style scoped>

.only-when-print {
    display: none;
    visibility: hidden;
}

@supports (display: grid) {
  .vertical {
    display: grid;
    grid-template-columns: max-content;
    grid-template-areas: "head body";
  }
  .vertical thead {
    grid-area: head;
  }
  .vertical tbody {
    grid-area: body;
  }
}

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
    min-width: max-content;
    min-height: 1.5em; /* Set a minimum height for table cells */
}

.table-container {
  overflow-x: auto;
  width: 100%;
  display: flex;
  flex-direction: column;
}

table { 
    border-collapse: collapse; 
}
table td {
  border: 1px solid black; 
}
table th {
    text-wrap: false;
    
  border: 1px solid black;
  background-color: #ffc800;
  color: rgb(0, 0, 0);
  text-align: left;
}
table tr:nth-child(10n), tr:nth-child(10n-1), tr:nth-child(10n-2), tr:nth-child(10n-3), tr:nth-child(10n-4) {
  background-color: #d2d2d2;
  color: #000;
}



.highlight-row {
  background-color: #ffa2e577; 
}

.highlight-col {
  background-color: #00ffbb33; 
}

@media (preffers-color-scheme: light) {
  .highlight-row {
    background-color: #ff77d877; 
  }

  .highlight-col {
    background-color: #1ff0b833; 
  }
}

@media print {
    .only-when-print {
        display: block;
        visibility: visible;
    }
    table {
        page-break-inside: auto;
        font-size: 8pt;
        background-color: #ffffff;
        color: #000000;
        border-collapse: separate;
        border-spacing: 0;
        margin: 0; /* Adjust margins to be stretched */
        width: 100%; /* Ensure the table stretches to full width */
    }

    thead {
        display: table-header-group; /* Ensure thead repeats on each page */
        position: sticky;
    }
    tbody tr {
        page-break-inside: avoid; /* Allow rows to break across pages */
    }
    th, td {
        border: 0.5px solid black;
        background-color: #ffffff;
        color: #000000;
    }
    .eliminar-columnas {
        visibility: hidden;
    }

}
.eliminar-columnas {
    background-color: #ffc1c100;
    color : #ffffff;
    height: 2.5em;
    /* centrar texto */
    display: flex;
    align-items: center;

}
.boton-eliminar-columnas {
    background-color: #ff0000;
    color: #ffffff;
    height: 2em;
    /* centrar */
    display: block;
    margin: auto;
    align-items: center;
    justify-content: center;
}
</style>
