<script setup>  
import { ref, computed, onMounted, watchEffect } from 'vue';
import { Icon } from '@iconify/vue';
const props = defineProps({
    totalItems: {
        type: Number,
        required: true
    },
    pageSize: {
        type: Number,
        default: 10
    },
    page: {
        type: Number,
        default: 0
    },
    query: {
        type: String
    },
    btnClass: {
        type: String
    },
    class: {
        type: String
    },
    nextIcon: String,
    maxNextIcon: String
})
const model = defineModel()
const currentPage = ref(1);
const route = useRoute()
const router = useRouter()
const totalPages = computed(() => Math.ceil(props.totalItems / props.pageSize));
const isFirstPage = computed(() => currentPage.value === 1);
const isLastPage = computed(() => currentPage.value === totalPages.value);
const firstPage = () => currentPage.value = 1;
const lastPage = () => currentPage.value = totalPages.value;
const nextPage = () => {
    if (currentPage.value < totalPages.value) currentPage.value += 1;
};
const previousPage = () => {
    if (currentPage.value > 1) currentPage.value -= 1;
};
watchEffect(() => {
    model.value = currentPage.value
    router.push({query: {albumId: currentPage.value} })
})
onMounted(() => {
    if (route.query[props.query]) {
        currentPage.value = Number(route.query[props.query])
        return
    }
    if (props.page) currentPage.value = props.page;
})
  </script>
<template>
    <div class="Paginator" :class="class">
        <button type="button" @click="firstPage" :disabled="isFirstPage" :class="btnClass">
            <Icon :icon="maxNextIcon || 'meteor-icons:angles-left'" width="20" height="20" />
        </button>
        <button type="button" @click="previousPage" :disabled="isFirstPage" :class="btnClass">
            <Icon :icon="nextIcon || 'meteor-icons:angle-left'" width="20" height="20" />
        </button>
        <span>({{ currentPage }} из {{ totalPages }})</span>
        
        <button type="button" @click="nextPage" :disabled="isLastPage" :class="btnClass">
            <Icon :icon="nextIcon || 'meteor-icons:angle-left'" width="20" height="20" />
        </button>
        <button type="button" @click="lastPage" :disabled="isLastPage" :class="btnClass">
            <Icon :icon="maxNextIcon || 'meteor-icons:angles-left'" width="20" height="20" />
        </button>
    </div>
</template>
  
<style scoped>
.Paginator {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
}
.Paginator button:nth-child(4),
.Paginator button:nth-child(5) {
    transform: rotate(180deg);
}
</style>
  