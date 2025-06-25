<template>
  <div class="page">
    <div class="scanner-card">
      <h1>🛡️ AntiVirus Scanner</h1>

      <div class="section">
        <label class="upload-label" @click="triggerFileInput">
          📁 Перетащите файл или нажмите здесь
          <input type="file" ref="fileInput" style="display: none" @change="handleFileChange" />
        </label>
      </div>

      <button class="scan-button" @click="startUpload" :disabled="!uploadedFile">
        Загрузить файл
      </button>

      <div class="progress" v-if="isLoading">⏳ Проверка файла...</div>

      <!-- Отображение загруженного файла -->
      <div class="file-preview" v-if="uploadedFile && !isLoading">
        <div class="file-info">
          📄 <strong>{{ uploadedFile.name }}</strong>
          <span class="size">({{ formatSize(uploadedFile.size) }})</span>
        </div>
        <button class="remove-btn" @click="removeFile">❌ Удалить</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
onMounted(() => {
  document.body.style.overflow = "hidden";
})

const fileInput = ref(null);
const uploadedFile = ref(null);
const isLoading = ref(false);
function triggerFileInput() {
  fileInput.value.click();
}
function handleFileChange() {
  const file = fileInput.value.files[0];
  if (file) {
    uploadedFile.value = file;
  }
}
function startUpload() {
  if (!uploadedFile.value) return;
  isLoading.value = true;
  }
function removeFile() {
  uploadedFile.value = null;
  fileInput.value.value = ""; // очистить input
}
function formatSize(bytes) {
  if (bytes < 1024) return `${bytes} Б`;
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} КБ`;
  return `${(bytes / 1024 / 1024).toFixed(1)} МБ`;
}
</script>
<style scoped>
.page {
  min-height: 100vh;
  width: 100vw;
  padding: 0;
  margin: 0;
  background: #f0f4f8;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow-x: hidden;
  overflow-y: hidden;
}

.scanner-card {
  background: #ffffff;
  width: 400px;
  padding: 40px 30px;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
  text-align: center;
  font-family: "Segoe UI", sans-serif;
}

.scanner-card h1 {
  font-size: 24px;
  margin-bottom: 30px;
  color: #2c3e50;
}

.section {
  margin-bottom: 20px;
}

.upload-label {
  display: block;
  border: 2px dashed #bbb;
  padding: 20px;
  border-radius: 10px;
  background: #fafafa;
  color: #333;
  cursor: pointer;
  transition: all 0.3s;
}
.upload-label:hover {
  background: #f0fff0;
  border-color: #4caf50;
}

.scan-button {
  width: 100%;
  padding: 12px;
  background-color: #4c6ef5;
  color: white;
  font-size: 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}
.scan-button:disabled {
  background-color: #aab5f9;
  cursor: not-allowed;
}
.scan-button:hover:enabled {
  background-color: #3b5bdb;
}

.progress {
  margin-top: 20px;
  font-size: 14px;
  color: #555;
}

.file-preview {
  margin-top: 20px;
  background: #eef3ff;
  border-left: 4px solid #4c6ef5;
  border-radius: 6px;
  padding: 12px 15px;
  text-align: left;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 14px;
  color: #2c3e50;
}
.file-info {
  display: flex;
  flex-direction: column;
}
.size {
  font-size: 12px;
  color: #666;
  margin-top: 2px;
}
.remove-btn {
  background: transparent;
  border: none;
  color: #e53935;
  font-size: 16px;
  cursor: pointer;
}
.remove-btn:hover {
  text-decoration: underline;
}
</style>

