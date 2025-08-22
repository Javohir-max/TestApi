<script setup>
import { ref } from 'vue'
import axios from 'axios'

const photos = ref([])
const selectedFiles = ref([])
const apiBase = 'https://me-backend-o4yi.onrender.com' // Замени на Render URL
const temporaryFile = ref([])
const dis = ref(false)
const photoName = ref([])
let intervalId = null
const loading = ref({
  now: 0,
  max: 0,
  active: false,
  wid: 0
})

// Когда выбираем файл
function handleFileUpload(event) {  
  const files = event.target.files
  if (!files) return
  dis.value = true
  selectedFiles.value = files
  loading.value.max = selectedFiles.value.length
  for (let i = 0; i < files.length; i++) {
    photoName.value.push(files[i].name.substring(0, files[i].name.lastIndexOf(".")))
    temporaryFile.value.push(URL.createObjectURL(files[i]))  
  }
  dis.value = false
}

// Отправка фото
async function uploadPhoto(file, name) {
  if (!file) return alert('Выбери файл')
  dis.value = true
  let count = 0
  intervalId = setInterval(() => {
    loading.value.now = count
    loading.value.max = file.length
    loading.value.wid = (count / file.length) * 100
    if (count === file.length) {
      clearInterval(intervalId)
      selectedFiles.value = []
      temporaryFile.value = []
      photoName.value = []
      dis.value = false
      setTimeout(() => {
        loading.value.active = false
      }, 500)
      fetchPhotos() 
    }else {
      postPhoto(file[count], name[count])
      loading.value.active = true
      count++
    }
  }, 1000)
}
async function postPhoto(file, name) {
  const formData = new FormData()
  formData.append('image', file)
  formData.append('name', name);
  try {
    await axios.post(`${apiBase}/photos`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    })
  } catch (error) {
    console.log(error);
    alert('Ошибка при загрузке')
    dis.value = false
  }
}

// Получение фото
async function fetchPhotos() {
  try {
    const res = await axios.get(`${apiBase}/photos`)
    photos.value = res.data    
  } catch (err) {
    console.error(err)
    alert('Ошибка при получении фото')
  }
}
async function delet(id) {
  dis.value = true
  try {
    await axios.delete(`${apiBase}/photos/${id}`)
    await fetchPhotos()
    alert('Успешно удаленно')
    dis.value = false
  } catch (error) {
    console.log(error);
    alert('Ошибкак при удалении!')
    dis.value = false
  }
}
onMounted(() => {
  fetchPhotos()
})
</script>
<template>
  <div>
    <div class="container">
      <!-- Загрузка -->
      <form @submit.prevent="uploadPhoto(selectedFiles, photoName)">
        <div class="load-box" :class="{'load-def': !loading.active}">
          <div class="loadding">
            <div class="back-fon" :style="{width: `${loading.wid}%`}"></div>
            <h2 class="load-text">
              {{ loading.now }} из {{ loading.max }}
            </h2>
          </div>
        </div>
        <div class="w-full flex flex-wrap">
          <img v-for="(item, index) in temporaryFile" :key="index" :src="item" style="width: 50px;" alt="foto">
        </div>
        <input type="file" @change="handleFileUpload" hidden id="image" name="file" multiple>
        <label for="image">+ Add Photo</label>
        <input type="text" v-model="photoName" name="photoname" placeholder="Названия Фото" required>
        <button type="submit" :class="{'btn-disabled': dis}">Загрузить</button>
      </form>
      <NuxtLink to="/">Index</NuxtLink>
      <NuxtLink to="/params">Query Params</NuxtLink>
      <!-- Список фото -->
      <div v-if="photos.length">
        <h2>Галерея</h2>
        <div style="display: flex; flex-wrap: wrap; gap: 10px;">
          <div
            v-for="(p, i) in photos"
            :key="i"
            class="box"
          >
            <img
              :src="p.url"
              alt="Фото"
              style="width: 200px; height: auto;"
            >
            <h2>{{ p.name }}</h2>
            <button @click="delet(p.id)" :class="{'btn-disabled': dis}">Delete</button>
          </div>
        </div>
      </div>
    </div>
    <NuxtPage />
  </div>
  <!-- <QueryParams /> -->
</template>
<style scoped>
  .container {
    width: 100%;
    /* height: 100vh; */
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    gap: 20px;
    margin: 0 auto;
  }
  form {
    width: 300px;
    min-height: 400px;
    outline: 1px solid red;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: end;
    position: relative;
    gap: 10px;
  }
  .load-box {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    backdrop-filter: blur(10px);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1;
  }
  .load-box .loadding {
    width: 150px;
    height: 20px;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 10px;
    position: relative;
    background-color: gray;
  }
  .load-box .back-fon {
    position: absolute;
    top: 0;
    left: 0;
    width: 0;
    height: 100%;
    background-color: green;
    border-radius: 10px;
    transition: .3s;
  }
  .load-box .load-text {
    color: white;
    position: relative;
    z-index: 2;
  }
  .load-def {
    display: none;
  }
  form img {
    outline: 1px solid red;
    text-align: center;
  }
  form label {
    padding: 5px 12px;
    background-color: dodgerblue;
    color: white;
    border-radius: 8px;
  }
  form input[type="text"] {
    border: none;
    background-color: beige;
    width: 70%;
    padding: 8px 16px;
    border-radius: 20px;
    text-align: center;
  }
  form input[type="text"]:focus {
    outline: 1px solid green;
  }
  form button {
    padding: 8px 16px;
    color: white;
    background-color: green;
    border: none;
    border-radius: 8px;
  }
  .box {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
  }
  .box button {
    padding: 8px 16px;
    background-color: red;
    border: none;
    color: white;
    border-radius: 8px;
  }
  button {
    cursor: pointer;
    transition: .3s;
  }
  .btn-disabled {
    opacity: .6;
  }
</style>
