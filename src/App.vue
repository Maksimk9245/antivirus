<template>
  <button @click="openDropzone" class="upload-btn">Upload</button>
    <div
      class="dropzone-content"
      :class="{ dragover:isDragOver }"
      @click="triggerFileInput"
      @dragover.prevent="onDragOver"
      @dragleave="onDragLeave"
      @drop.prevent="onDrop"
    >Сюда файл
    <p>
      Перетянуть файл<br />
      <span>Кликни, что бы выбрать с диска</span>
    </p>
    </div>
  <input type="file"
         ref="fileInput"
         style="display:none"
         @change="onFileChange"/>
  <div v-if="uploadedFile" class="uploaded-info"
  >Загруженно:<strong>{{uploadedFile.name}}</strong>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const isDragOver = ref(false)
const uploadedFile = ref(null)
const fileInput = ref(null)

function triggerFileInput() {
  fileInput.value?.click()
}

function onFileChange(e) {
  const file = e.target.files[0]
  if (file) {
    uploadedFile.value = file
    console.log('Выбран файл:', file)
  }
}

function onDragOver() {
  isDragOver.value = true
}

function onDragLeave() {
  isDragOver.value = false
}

function onDrop(e) {
  const file = e.dataTransfer.files[0]
  if (file) {
    uploadedFile.value = file
    console.log('Файл перетащен:', file)
  }
  isDragOver.value = false
}
</script>

<style scoped>
.upload-btn {
  background-color: #4caf50;
  color: white;
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  margin-bottom: 20px;
  transition: background-color 0.2s ease;
}
.upload-btn:hover {
  background-color: #45a049;
}

.dropzone-content {
  border: 2px dashed #aaa;
  padding: 40px;
  text-align: center;
  border-radius: 10px;
  background-color: #fff;
  cursor: pointer;
  transition: 0.2s ease;
  width: 100%;
  max-width: 400px;
  margin: 0 auto;
  color: #333;
}

.dropzone-content.dragover {
  border-color: #4caf50;
  background-color: #f0fff0;
}

.drop-text {
  font-size: 16px;
  line-height: 1.6;
}
.drop-text span {
  color: #777;
  font-size: 14px;
}

.uploaded-info {
  margin-top: 20px;
  font-size: 16px;
  padding: 12px;
  background-color: #f9f9f9;
  border-left: 4px solid #4caf50;
  border-radius: 5px;
  max-width: 400px;
  margin-left: auto;
  margin-right: auto;
}
</style>
