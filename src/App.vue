<script setup>
import HelloWorld from './components/HelloWorld.vue'
import FlujoNew from './components/FlujoNew.vue'
import IngresoLab from './components/IngresoLab.vue'
import IngresoLabNew from './components/IngresoLabNew.vue'

// print content on FlujoNew
const print = () => {
  // open new page with table content and print with print.css styling

  const printContent = document.getElementById('table-container').outerHTML;
  const originalContent = document.body.innerHTML;

  document.body.innerHTML = `${printContent}`;
  
  window.print();
  document.body.innerHTML = originalContent;
};


</script>

<template>
  <IngresoLabNew />
  <button @click="print" id="print-button">Imprimir flujo de exámenes</button>
  <FlujoNew />
  <div style="display: flex; 
    position: absolute; top: 0; right: 0; padding: 0px; height: 150px; background: #1cff86; border-radius: 5px;" class="only-desktop"
    >
        <p style="width: 175px; padding: 5px 5px 0px 5px; margin: 0px; color: rgb(0, 0, 0); font-size: 12pt; align-self: center;
        "><b>Si te fue útil, puedes ayudarme con mi adicción por el cafecito </b>en el siguiente código QR 🙂 (Atte. el interno informático)</p>
        <p style="width: 30px; padding: 0px; margin: 0px; color: rgb(0, 0, 0); font-size: 20pt; 
        align-self: center;  "
        >→</p>
        <img src="./assets/qr.png" style="height: 150px !important;width: 150px !important;">

    </div>
    
</template>

<style>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}



</style>
