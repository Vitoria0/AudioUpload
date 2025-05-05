<template>
  <div class="container">
    <div class="header">
      <h1>Audio Test</h1>
      <p class="subheading">Envie e ouça seus arquivos de áudio</p>
    </div>

    <form @submit.prevent="handleUpload" class="upload-form">
      <label for="fileInput" class="upload-label">
        <input type="file" id="fileInput" @change="onFileChange" />
        <span class="upload-btn">Escolher Arquivo</span>
      </label>
      <button :disabled="!selectedFile || loading" class="upload-button">
        <span v-if="loading">Uploading...</span>
        <span v-else>Upload</span>
      </button>
    </form>

    <div v-if="successMsg" class="success-msg">{{ successMsg }}</div>
    <div v-if="errorMsg" class="error-msg">{{ errorMsg }}</div>

    <div class="content">
      <!-- Reprodutor de Áudio -->
      <div class="audio-player-container">
        <h2>Reproduzir Áudio</h2>
        <audio v-if="selectedAudio" :src="'http://localhost:5000/files/' + selectedAudio" controls class="audio-player">
          Seu navegador não suporta o elemento de áudio.
        </audio>
      </div>

      <!-- Listagem de Arquivos -->
      <div class="file-list-container">
        <h2>Arquivos Enviados:</h2>
        <ul class="file-list">
          <li v-for="file in files" :key="file.filename" class="file-item" @click="selectAudio(file.filename)">
            <div class="file-info">
              <strong>{{ file.filename }}</strong> - {{ file.type }} - {{ formatDate(file.upload_date) }}
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const selectedFile = ref(null)
const selectedAudio = ref(null) // Para armazenar o arquivo de áudio selecionado
const files = ref([])
const loading = ref(false)
const successMsg = ref('')
const errorMsg = ref('')

const onFileChange = (e) => {
  selectedFile.value = e.target.files[0]
  successMsg.value = ''
  errorMsg.value = ''
}

const handleUpload = async () => {
  if (!selectedFile.value) return
  loading.value = true

  const formData = new FormData()
  formData.append('file', selectedFile.value)

  try {
    const res = await fetch('http://localhost:5000/upload', {
      method: 'POST',
      body: formData,
    })

    const result = await res.json()

    if (!res.ok) {
      throw new Error(result.error || 'Upload failed')
    }

    successMsg.value = 'Arquivo enviado com sucesso!'
    selectedFile.value = null
    await fetchFiles()
  } catch (err) {
    errorMsg.value = err.message
  } finally {
    loading.value = false
  }
}

const fetchFiles = async () => {
  try {
    const res = await fetch('http://localhost:5000/files')
    files.value = await res.json()
  } catch (err) {
    errorMsg.value = 'Erro ao buscar arquivos'
  }
}

const formatDate = (iso) => {
  const date = new Date(iso)
  return date.toLocaleString()
}

const selectAudio = (filename) => {
  selectedAudio.value = filename // Atualiza o áudio selecionado
}

onMounted(fetchFiles)
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Arial', sans-serif;
  background-color: #f4f7fc;
  color: #333;
}

.container {
  max-width: 1200px;
  margin: 2rem auto;
  background: #fff;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

h1 {
  font-size: 2rem;
  color: #4caf50;
  margin-bottom: 0.5rem;
}

.subheading {
  font-size: 1rem;
  color: #888;
}

.upload-form {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.upload-label {
  display: inline-block;
  background-color: #4caf50;
  padding: 10px 20px;
  color: #fff;
  border-radius: 4px;
  cursor: pointer;
  margin-bottom: 1rem;
}

.upload-btn {
  font-size: 16px;
}

.upload-input {
  display: none;
}

.upload-button {
  padding: 10px 20px;
  background-color: #4caf50;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

.upload-button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.success-msg {
  background-color: #28a745;
  color: white;
  padding: 10px;
  margin: 1rem 0;
  border-radius: 4px;
}

.error-msg {
  background-color: #dc3545;
  color: white;
  padding: 10px;
  margin: 1rem 0;
  border-radius: 4px;
}

.content {
  display: flex;
  gap: 2rem;
  margin-top: 2rem;
}

.audio-player-container {
  flex: 1;
  max-width: 600px;
  text-align: center;
}

.audio-player {
  width: 100%;
  border-radius: 4px;
  margin-top: 10px;
}

.file-list-container {
  flex: 1;
  max-width: 400px;
}

.file-list {
  list-style-type: none;
  padding: 0;
}

.file-item {
  background-color: #f9f9f9;
  margin: 10px 0;
  padding: 10px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.file-info {
  font-size: 14px;
  color: #555;
}

.file-item:hover {
  background-color: #e0e0e0;
}

@media (max-width: 900px) {
  .content {
    flex-direction: column;
    gap: 1rem;
  }

  .audio-player-container, .file-list-container {
    max-width: 100%;
  }
}

@media (max-width: 600px) {
  .container {
    width: 90%;
    padding: 1.5rem;
  }

  .upload-button {
    font-size: 14px;
  }
}
</style>
