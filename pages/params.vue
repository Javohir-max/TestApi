<script setup>
import { computed, onMounted, ref } from 'vue'
import CompPaginator from '../components/CompPaginator.vue'
import axios from 'axios'
const jsonUrl = "https://jsonplaceholder.typicode.com/photos"
const jsonData = ref([])

const pagLen = ref(null)

const filter = computed(() => {
  return (jsonData.value || []).filter(fl => fl.albumId === pagLen.value)
})

const getObject = async () => {
  try {
    const { data } = await axios(jsonUrl)
    jsonData.value = data || []
  } catch (error) {
    console.log(error)
  }
}
onMounted(() => {
  getObject()
})
</script>
<template>
    <div class="query_params">
        <div class="query_box">
            <div class="box_container">
                <div v-for="(item, index) in filter" :key="index" class="box">
                    {{ item.id}} 
                </div>
            </div>
            <CompPaginator 
                :totalItems="jsonData.length" 
                v-model="pagLen"
                :pageSize="50"
                query="albumId"
            />
        </div>
    </div>
</template>
<style lang="scss">
    .query_params {
        width: 100%;
        height: 100vh;
        display: flex;
        justify-content: center;
        align-items: center;
        .query_box {
            width: 80%;
            height: 900px;
            border: 1px solid red;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 25px;
            .box_container {
                width: 100%;
                height: 800px;
                overflow: auto;
                display: flex;
                flex-wrap: wrap;
                justify-content: start;
                gap: 20px;
                border: 1px solid blue;
                .box {
                    width: 300px;
                    height: 300px;
                    border: 1px solid green;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                }
            }
        }
    }
</style>