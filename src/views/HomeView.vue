<template>
    <header>
        <nav class="bg-white border-gray-200 px-4 lg:px-6 py-2.5 dark:bg-gray-800">
            <div class="flex flex-wrap justify-between items-center mx-auto max-w-screen-xl">
                <a href="https://flowbite.com" class="flex items-center">
                    <img
                        src="https://www.atlansun.fr/uploads/media/page_membre_equipe/05/215-SEE_YOU_SUN.jpg?v=5-0"
                        class="mr-3 h-6 sm:h-9"
                        alt="Flowbite Logo"
                    />
                    <span
                        class="self-center text-xl font-semibold whitespace-nowrap dark:text-white"
                        >SeeYouSun</span
                    >
                </a>
                <div class="flex items-center">
                  <span class="pr-12 text-white">Site de Production : Spie Carquefou</span>

                    <span class="pr-2 text-white"> période en cours : </span>
                    <VueDatePicker
                        id="datePicker"
                        @range-start="onRangeStart"
                        @range-end="onRangeEnd"
                        v-model="date"
                        :dark="true"
                        :enable-time-picker="false"
                        range
                    >
                    </VueDatePicker>
                </div>
            </div>
        </nav>
    </header>
    <main>
        <AeraGraphView :date="newDate" @brush-date="brushDate" />
    </main>
</template>

<script setup>
import AeraGraphView from '../components/AeraGraphView.vue'
import { ref, onMounted } from 'vue'

let startDate = new Date()
let endDate = new Date(new Date().setDate(startDate.getDate() + 7))
const date = ref([])
let newDate = ref([])

const onRangeStart = (value) => {
    date.value[startDate] = value
}

const onRangeEnd = (value) => {
    date.value[endDate] = value
    newDate.value = [date.value[startDate], date.value[endDate]]
}

const brushDate = (value) =>{
    date.value = value
}

onMounted(() => {
  date.value = [startDate, endDate]
})
</script>

<style>
body {
    margin: 0;
    padding: 0;
}
</style>
