<script>

export default {
    data() {
        return {
            fileNames: [],
        };
    },
  methods: {
    handleFileChange(event) {
      const files = event.target.files;
        this.fileNames = [];
        for (let i = 0; i < files.length; i++) {
            this.fileNames.push(files[i].name);
        }
    },
  },
};


</script>

<template>
  <h1>Ingreso de lab</h1>
  <div class="container">
    <div class="container-ingresos">
        <label for="link">Ingrese link aquí: </label>
        <input type="text" id="link" name="link">
        <button>Añadir link a flujo de exámenes</button>
        
    </div>
    <div class="container-ingresos">
        <label for="archivo">O suba los archivos aquí: </label>
        
        <input type="file" id="archivo" name="archivo" accept="application/pdf" multiple ref="input" @change="handleFileChange" />
        <p>Nombres de archivos: </p>
        <ol>
            <li v-for="(file, index) in fileNames" :key="index"">
                {{ file }}
            </li>
        </ol>
        <button>Añadir archivos a flujo de examenes</button>
    </div>
    </div>
    <br><br>
  

</template>

<style scoped>

.container {
  display: flex;
  flex-direction: row;
  align-items: top;
}
.container-ingresos {
    display: flex;
    flex-direction: column;
    align-items: left;
    margin: 1em;
    border: 1px solid black;
    border-radius: 1em;
    background-color: #75e8ff39;
    padding: 1em;
}

input{
    margin: 0.5em;
    text-align: left;
    /* 
    on top */
    vertical-align: top;

}

</style>