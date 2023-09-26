<script setup lang="ts">
import { proxyRefs, ref } from "vue";
import tj from "@mapbox/togeojson";
import toKml from "tokml";
import FileSaver from "file-saver";

defineProps<{ msg: string }>();

//可以被转换
const isReady = ref(false);

//正在转换
const isConverting = ref(false);

let file: File | null = null;

const fileInput = ref(null);

function handleFileInputChange(e: Event) {
  const target = e.target as HTMLInputElement;
  if (target.files) {
    file = target.files[0];
    console.log(target.files[0]);
    isReady.value = true;
  }
}

//开始转换
function startConvert() {
  if (file == null) {
    return;
  }

  if (isConverting.value) {
    return;
  }

  isConverting.value = true;
  const reader = new FileReader();
  reader.onload = function (e) {
    var text: string = reader.result as string;
    var gpx = new DOMParser().parseFromString(text, "text/xml");
    let geojson = tj.gpx(gpx);
    let kml = toKml(geojson);
    var blob = new Blob([kml], { type: "text/plain;charset=utf-8" });
    FileSaver(blob, file!.name.replace(".gpx", ".kml"));
    file = null;
    isConverting.value = false;
    fileInput.value = null;
  };

  reader.readAsText(file);
}
</script>

<template>
  <h1>{{ msg }}</h1>
  <div class="body">
    <input type="file" :value="fileInput" @change="handleFileInputChange" />

    <button @click="startConvert" :disabled="!isReady">
      <div
        style="
          display: flex;
          align-items: center;
          justify-content: center;
          gap: 0.5rem;
        "
      >
        <span>开始转换</span>
        <span class="btn-ring" v-show="isConverting"></span>
      </div>
    </button>
  </div>
</template>

<style scoped>
.body {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.read-the-docs {
  color: #888;
}

button {
  margin: auto auto;
}

botton:disabled {
  background: gray;
}

.btn-ring {
  content: "";
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  border: 3px solid #fff;
  border-color: #fff transparent #fff transparent;
  animation: ring 1.2s linear infinite;
}
@keyframes ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
