<template>
  <div class="page">
    <div class="content">
      <div class="scanner-card">
        <h1>🛡️ AntiVirus Scanner (VirusTotal)</h1>
        <div class="section">
          <label class="upload-label" @click="triggerFileInput">
            📁 Перетащите файл или нажмите здесь
          </label>
          <input type="file" ref="fileInput" style="display: none" @change="handleFileChange" />
        </div>
        <button class="scan-button" @click="startUpload" :disabled="!uploadedFile || isLoading">
          Загрузить файл
        </button>
        <div class="progress" v-if="isLoading">{{ resultMessage }}</div>
        <div class="result" v-if="resultMessage && !isLoading">{{ resultMessage }}</div>
        <div class="file-preview" v-if="uploadedFile && !isLoading">
          <div class="file-info">
            📄 <strong>{{ uploadedFile.name }}</strong>
            <span class="size">({{ formatSize(uploadedFile.size) }})</span>
          </div>
          <button class="remove-btn" @click="removeFile">❌ Удалить</button>
        </div>
        <div v-if="analysisData" class="analysis-details">
          <h3>Детали анализа:</h3>
          <pre>{{ JSON.stringify(analysisData, null, 2) }}</pre>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const API_KEY = import.meta.env.VITE_API_KEY;
const UPLOAD_URL = "https://www.virustotal.com/api/v3/files";
const FILE_INFO_URL = "https://www.virustotal.com/api/v3/files/";
const ANALYSIS_URL_BASE = "https://www.virustotal.com/api/v3/analyses/";

const fileInput = ref(null);
const uploadedFile = ref(null);
const isLoading = ref(false);
const resultMessage = ref("");
const analysisData = ref(null);

onMounted(() => {
  document.body.style.overflow = "hidden";
});

function triggerFileInput() {
  fileInput.value.click();
}

function handleFileChange() {
  const file = fileInput.value.files[0];
  if (file) {
    uploadedFile.value = file;
    resultMessage.value = "";
    analysisData.value = null;
  }
}

async function startUpload() {
  if (!uploadedFile.value) return;

  isLoading.value = true;
  resultMessage.value = "Загрузка файла на VirusTotal...";

  const formData = new FormData();
  formData.append("file", uploadedFile.value);

  try {
    const uploadResponse = await fetch(UPLOAD_URL, {
      method: "POST",
      headers: { "x-apikey": API_KEY },
      body: formData,
    });

    if (uploadResponse.status === 409) {
      const errorData = await uploadResponse.json();
      const fileHash = errorData.error?.file_hash || null;
      if (!fileHash) throw new Error("Нет хеша файла при 409");
      resultMessage.value = "Файл уже загружен. Получаем результаты анализа по хешу...";
      await getAnalysisByFileHash(fileHash);
      return;
    }

    if (!uploadResponse.ok) {
      throw new Error(`Ошибка загрузки файла: ${uploadResponse.status}`);
    }

    const uploadResult = await uploadResponse.json();
    const analysisId = uploadResult.data.id;
    if (!analysisId) throw new Error("Нет ID анализа в ответе");

    resultMessage.value = "Файл загружен. Ожидание результатов анализа...";
    await pollAnalysisResult(analysisId);

  } catch {
    resultMessage.value = "Ошибка при загрузке или анализе файла.";
  } finally {
    isLoading.value = false;
  }
}

async function getAnalysisByFileHash(fileHash) {
  try {
    const fileInfoResponse = await fetch(FILE_INFO_URL + fileHash, {
      headers: { "x-apikey": API_KEY },
    });
    if (!fileInfoResponse.ok) throw new Error(`Ошибка получения информации о файле: ${fileInfoResponse.status}`);

    const fileInfo = await fileInfoResponse.json();
    const analysisId = fileInfo.data?.attributes?.last_analysis_id;

    if (!analysisId) {
      resultMessage.value = "Нет информации об анализе для данного файла.";
      return;
    }

    resultMessage.value = "Получаем результаты анализа...";
    await pollAnalysisResult(analysisId);

  } catch {
    resultMessage.value = "Ошибка при получении информации о файле по хешу.";
  }
}

async function pollAnalysisResult(analysisId, attempts = 10, delayMs = 3000) {
  for (let i = 0; i < attempts; i++) {
    resultMessage.value = `Ожидание результатов анализа... попытка ${i + 1} из ${attempts}`;
    try {
      const response = await fetch(ANALYSIS_URL_BASE + analysisId, {
        headers: { "x-apikey": API_KEY },
      });

      if (!response.ok) {
        throw new Error(`Ошибка получения анализа: ${response.status}`);
      }

      const data = await response.json();
      const status = data.data.attributes.status;

      if (status === "completed") {
        analysisData.value = data;
        const stats = data.data.attributes.stats;
        resultMessage.value = `Анализ завершён. Обнаружено вредоносных: ${stats.malicious} из ${Object.values(stats).reduce((a, b) => a + b, 0)}.`;
        return;
      }
    } catch {
      resultMessage.value = "Ошибка при получении результата анализа.";
      return;
    }
    await new Promise((resolve) => setTimeout(resolve, delayMs));
  }
  resultMessage.value = "Время ожидания анализа истекло. Попробуйте позже.";
}

function removeFile() {
  uploadedFile.value = null;
  fileInput.value.value = "";
  resultMessage.value = "";
  analysisData.value = null;
}

function formatSize(bytes) {
  if (bytes < 1024) return `${bytes} Б`;
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} КБ`;
  if (uploadedFile.value.size > 32 * 1024 * 1024) {
    alert("Файл слишком большой. Максимальный размер — 32 МБ.");
    return;
  }
  return `${(bytes / 1024 / 1024).toFixed(1)} МБ`;

}
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow-x: hidden;
  box-sizing: border-box;
  background: white;
}
*,
*::before,
*::after {
  box-sizing: inherit;
}
.page {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  box-sizing: border-box;
  height: 100vh;
  justify-content: center;
  align-items: center;
}
.content {
  height: 100vh;
  width: 1200px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.scanner-card {
  width: 400px;
  background: navajowhite;
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
.result {
  margin-top: 20px;
  font-size: 16px;
  color: #2c3e50;
  font-weight: 600;
  white-space: pre-line;
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
.analysis-details {
  margin-top: 20px;
  max-height: 300px;
  overflow-y: auto;
  text-align: left;
  background: #f7f9fc;
  border: 1px solid #ccc;
  padding: 10px;
  border-radius: 6px;
  font-size: 12px;
  white-space: pre-wrap;
}
</style>
