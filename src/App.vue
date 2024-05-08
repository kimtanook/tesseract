<template>
  <div>
    <input type="file" @change="handleFileChange" />
    <div v-if="imageData">
      <img class="uploaded_image" :src="imageData" alt="Uploaded Image" />
      <button @click="recognizeText" :disabled="loading">텍스트 인식</button>
      <div v-if="loading">
        <progress :value="loadingPercent" min="0" max="100"></progress>
      </div>
      <p v-else-if="recognizedText">인식된 텍스트: {{ recognizedText }}</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import Tesseract from "tesseract.js";
import {ref} from "vue";

const imageData: any = ref(null);
const recognizedText: any = ref(null);
const loading = ref(false);
const loadingPercent = ref(0);

const handleFileChange = (event: any) => {
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onload = () => {
    imageData.value = reader.result;
  };
  reader.readAsDataURL(file);
};

const recognizeText = async () => {
  loading.value = true;
  try {
    const {
      data: {text},
    } = await Tesseract.recognize(imageData.value, "eng", {
      logger: (m) => {
        if (m.status == "recognizing text") {
          loadingPercent.value = m.progress.toFixed(2) * 100;
        }
      },
    });
    console.log("text : ", text);
    recognizedText.value = text.split("\n").filter((line) => line !== "");
  } catch (error) {
    console.error("텍스트를 인식하는 중 오류 발생:", error);
  } finally {
    loading.value = false;
  }
};
</script>
<style scoped>
.uploaded_image {
  width: 80vw;
}
</style>
