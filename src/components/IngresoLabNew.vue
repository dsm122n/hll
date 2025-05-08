<script setup>
import { ref } from 'vue';
import * as pdfjsLib from "pdfjs-dist";
import "pdfjs-dist/build/pdf.worker";
import { labResults } from "../stores/labResultsStore.js";
import { LabResultClass } from "../models/LabResultsClass.js";

const LabEquivalence = {
    // en pdf : en objeto para th tabla
        "Creatinina" : "Creatinina",
        "Lactato" : "Lactato",
        "Proteina C Reactiva" : "PCR",
        "Sodio ( Na )" : "Na",
        "Potasio ( K )" : "K",
        "Cloro ( Cl )" : "Cl",
        "proBNP" : "proBNP",
        "Calcio" : "Ca",
        "Uremia" : "Uremia",
        "Nitrogeno ureico" : "BUN",
        "Magnesio" : "Magnesio",
        "Fosforo" : "Fosforo",
        "Bilirrubina Total" : "Bili T",
        "Bilirrubina Directa" : "Bili D",
        "ASAT/GOT" : "ASAT/GOT",
        "ALAT/GPT" : "ALAT/GPT",
        "Gama GT" : "GamaGT",
        "Fosfatasas Alcalinas" : "Fosfatasas Alcalinas",
        "Troponina I" : "Troponina I",
        "LDH" : "LDH",
        "Albumina" : "Albumina",
        "Proteinas" : "Proteinas",
        "Glucosa" : "Glucosa",
        "Trigliceridos" : "Trigliceridos",
        "CK Total" : "CK Total",
        "CK MB" : "CK-MB",
        "V.H.S." : "VHS",
        "Hematocrito" : "Hto",
        "Hemoglobina" : "Hb",
        "Recuento Hematíes" : "Recuento Hematíes",
        "VCM" : "VCM",
        "HCM" : "HCM",
        "CHCM" : "CHCM",
        "Recuento Leucocitos" : "Leucocitos",
        "Recuento de Plaquetas" : "Plaquetas",
        "RDW" : "RDW",
        // "Segmentados %" : "Segmentados %",
        // "Linfocitos %" : "Linfocitos %",
        // "Monocitos %" : "Monocitos %",
        // "Basófilos %" : "Basófilos %",
        // "Eosinófilos %" : "Eosinófilos %",
        "Eosinófilos" : "Eosinófilos",
        "Basófilos" : "Basófilos",
        "Segmentados" : "Segmentados",
        "Linfocitos" : "Linfocitos",
        "Monocitos" : "Monocitos",
        "pH" : "pH",
        "pO2" : "pO2",
        "pCO2" : "pCO2",
        "HCO3-" : "HCO3",
        "BE (B)" : "BE",
        "sO2" : "sO2",
        "INR" : "INR",
        "Tiempo de protrombina %" : "TP%",
        "Tiempo de protrombina seg" : "TP seg",
        "TTPK" : "TTPK"
    
};

const loadPdf = async (file) => {
    try {
        const fileReader = new FileReader();
        fileReader.onload = async (e) => {
            const pdfData = e.target.result;
            const pdf = await pdfjsLib.getDocument({ data: pdfData }).promise;
            let extractedData = new LabResultClass();

            for (let i = 1; i <= pdf.numPages; i++) {
                const page = await pdf.getPage(i);
                const textContent = await page.getTextContent();
                const pageText = textContent.items.map((item) => item.str).join(" ");
                console.log("Page text:", pageText);
                if (i === 1) {
                    const regex = /Fecha y Hora Ingreso Solicitud\s*:\s*(\d{2}-\d{2}-\d{4})\s+(\d{2}:\d{2}:\d{2})/i;
                    const match = pageText.match(regex);
                    if (match) {
                        const [_, datePart, timePart] = match;
                        const [day, month, year] = datePart.split("-");
                        const [hours, minutes, seconds] = timePart.split(":");
                        const dateObject = new Date(year, month - 1, day, hours, minutes, seconds);
                        extractedData["Fecha"] = dateObject;
                    }
                }

                for (let examen of Object.keys(LabEquivalence)) {
                    const regex = new RegExp(`${examen.replace(/[()]/g, '\\$&')}\\s*[:]\\s*(\\d+[.,]?\\d*\\s*\\*?)`, "i");
                    const match = pageText.match(regex);
                    if (match) {
                        extractedData[LabEquivalence[examen]] = match[1].replace(/\s/g, '');
                    }
                }
            }

            // Add the extracted data to a temporary array
            const tempResults = [...labResults.value, {
                fileName: file.name,
                extractedData,
            }];

            // Sort the temporary array by date
            tempResults.sort((a, b) => a.extractedData.Fecha - b.extractedData.Fecha);

            // Update the labResults store with the sorted data
            labResults.value = tempResults;
        };

        fileReader.readAsArrayBuffer(file);
    } catch (error) {
        console.error("Error al cargar el PDF:", error);
    }
};

const importPDFArchivo = () => {
    const fileInput = document.getElementById("archivo");
    if (fileInput.files.length === 0) {
        console.error("No se han seleccionado archivos");
        return;
    }

    for (let i = 0; i < fileInput.files.length; i++) {
        loadPdf(fileInput.files[i]);
    }
};

const importPDF = () => {
    const linkInput = document.getElementById("link");
    const link = linkInput.value;
    if (!link) {
        console.error("No se ha ingresado un link");
        return;
    }

    const fileName = link.split("/").pop();
    fetch(link)
        .then((response) => response.blob())
        .then((blob) => {
            const file = new File([blob], fileName, { type: "application/pdf" });
            loadPdf(file);
        })
        .catch((error) => {
            console.error("Error al cargar el archivo:", error);
        });
};

const fileNames = ref([]);

const handleFileChange = (event) => {
    const files = event.target.files;
    fileNames.value = [];
    for (let i = 0; i < files.length; i++) {
        fileNames.value.push(files[i].name);
    }
};

</script>

<template>
    <h1>Ingreso de datos</h1>
    <div class="container">
        <div class="container-ingresos">
            <label for="link">Ingrese link aquí: </label>
            <input type="text" id="link" name="link">
            <button @click="importPDF" id="boton-url">Añadir link a flujo de exámenes</button>
        </div>
        <div class="container-ingresos">
            <label for="archivo">O suba los archivos aquí: </label>
            <input type="file" id="archivo" name="archivo" accept="application/pdf" multiple @change="handleFileChange" />
            <p>Nombres de archivos: </p>
            <ol>
                <li v-for="(file, index) in fileNames" :key="index">
                    {{ file }}
                </li>
            </ol>
            <button @click="importPDFArchivo" id="boton-archivo" >Añadir archivos a flujo de exámenes</button>
        </div>
        <div>
            <h1>Cómo usar</h1>
    <p>Para añadir un archivo PDF al flujo de exámenes, puedes:</p>
    <ul>
        <li><span class="highlight">Ingresar el link del SysLab</span> que aparece cuando abres un examen:
            <ol>
                <li>Copia y pega el link en el campo de texto.</li>
                <li>Haz click en el botón <span class="highlight">"Añadir link a flujo de exámenes"</span>.</li>
            </ol>
        </li>
        <li><span class="highlight">Subir el archivo PDF directamente</span>:
            <ol>
                <li>Haz click en el botón <span class="highlight">"Seleccionar archivo"</span> o arrastra uno o varios archivos PDF al recuadro central.</li>
                <li>Selecciona uno o varios archivos PDF.</li>
                <li>Haz click en el botón <span class="highlight">"Añadir archivos a flujo de exámenes"</span>.</li>
            </ol>
        </li>
    </ul>
    <p>Puedes <span class="funcionalidad">añadirlos</span> de forma individual o en grupo. Los archivos se ordenarán por fecha del examen.<br>
    Si deseas <span class="funcionalidad">eliminar</span> un día de laboratorio, puedes hacerlo con el botón rojo <button class="boton-eliminar-columnas"></button> 
    desde la tabla de resultados.<br>
    Luego puedes <span class="funcionalidad">imprimir</span> la tabla de resultados con el botón <span class="highlight">"Imprimir flujo de exámenes"</span>. <br>
        Al final de la tabla incluí unos espacios en blanco para que puedas agregar exámenes adicionales.
</p>
    

    <p><span class="important">Nota:</span> todavía tiene un par de <span class="important">errores</span>, por ejemplo:</p>
    <ul>
        <li>No puede imprimir más de una página :( habría que ingresar de a poco los exámenes para que no superen la página.</li>
        <li>Algunos exámenes no los lee (por ejemplo, TP seg y TP%).</li>
        <li>Si subes examenes de orina u otros fluidos, puede ser que interprete algunos valores (por ejemplo, podría poner Leucocitos urinarios en el campo de Leucos destinado a leucos de sangre, lo mismo para el pH urinario, neutrofilos en liquido peritoneal, etc)</li>
    </ul>

    <p>Si pillas otro error, o tienes sugerencias para mejorar este programita avísame por favor! -> <a href="mailto:dsm122n@gmail.com">dsm122n@gmail.com</a></p>. El codigo está disponible en <a href="https://github.com/dsm122n/flujo_lab">GitHub</a>.
        </div>
    </div>
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
input {
    margin: 0.5em;
    text-align: left;
    vertical-align: top;
}
button {
    margin-left: 0.5em;
    padding: 0.2em 0.5em;
    cursor: pointer;
}
button:disabled {
    background-color: #ddd;
    cursor: not-allowed;
}
input[type="file"] {
    width: 200px;
    height: 100px;
    background-color: #75e8ff39;
    border-radius: 1em;
    align-items: center;
    justify-content: center;
    padding: 1em;
}
/* center #archivo button */
#file-upload-button {
    justify-content: center;
    align-items: center;
    height: 100%;
}
.boton-eliminar-columnas {
    background-color: #ff0000;
    color: #ffffff;
    height: 2em;
    width: 2em;
    /* centrar */

}
</style>
