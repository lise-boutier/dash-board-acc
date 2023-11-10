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
                    <!-- <a
                        href="#"
                        class="text-gray-800 dark:text-white hover:bg-gray-50 focus:ring-4 focus:ring-gray-300 font-medium rounded-lg text-sm px-4 lg:px-5 py-2 lg:py-2.5 mr-2 dark:hover:bg-gray-700 focus:outline-none dark:focus:ring-gray-800"
                        >Log in</a
                    >
                    <a
                        href="#"
                        class="text-white bg-primary-700 hover:bg-primary-800 focus:ring-4 focus:ring-primary-300 font-medium rounded-lg text-sm px-4 lg:px-5 py-2 lg:py-2.5 mr-2 dark:bg-primary-600 dark:hover:bg-primary-700 focus:outline-none dark:focus:ring-primary-800"
                        >Get started</a
                    > -->
                </div>
            </div>
        </nav>
    </header>
    <div class="mt-6 p-6" id="my_dataviz"></div>
</template>

<script setup>
import { onMounted, ref,onUpdated } from 'vue'
import * as d3 from 'd3'


let newArray = []

// set the dimensions and margins of the graph
const margin = { top: 10, right: 30, bottom: 30, left: 60 },
    width = 1200 - margin.left - margin.right,
    height = 550 - margin.top - margin.bottom

let startDate = new Date()
let endDate = new Date(new Date().setDate(startDate.getDate() + 7))
let date = ref()
let svg = d3
    .select('#my_dataviz')
    .append('svg')
    .attr('width', width + margin.left + margin.right)
    .attr('height', height + margin.top + margin.bottom)
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`)
let x
let y = d3
    .scaleLinear()
    .domain([0, d3.max(newArray, (d) => +d.value)])
    .range([height, 0])

let xAxis

let areaGenerator = d3.area()

// Create an area generator

function dateChartUpdate(date) {
    x.domain([date[startDate], date[endDate]])
    // Update axis and area position
    xAxis.transition().duration(1000).call(d3.axisBottom(x))
    svg.select('.myArea').transition().attr('d', areaGenerator)
}

const onRangeStart = (value) => {
    date.value[startDate] = value
}

const onRangeEnd = (value) => {
    date.value[endDate] = value
    dateChartUpdate(date.value)
}

function initGraph() {
    // append the svg object to the body of the page
    svg = d3
        .select('#my_dataviz')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)
        .append('g')
        .attr('transform', `translate(${margin.left},${margin.top})`)

    d3.json('http://localhost:8000/daily_prod_dur').then((data) => {
        let format = JSON.parse(data[0].production.replace(/'/g, '"'))
        for (let i = 0; i < format.length; i++) {
            newArray.push({
                date: d3.timeParse('%Y-%m-%d')(format[i].timestamp),
                value: format[i].prod_kwh
            })
        }

        // Add X axis --> it is a date format
        x = d3
            .scaleTime()
            .domain(d3.extent(newArray, (d) => d.date))
            .range([0, width])
        xAxis = svg
            .append('g')
            .attr('transform', `translate(0,${height})`)
            .call(d3.axisBottom(x))
            .attr('id', 'axisRed')

        // Add Y axis
        y = d3
            .scaleLinear()
            .domain([0, d3.max(newArray, (d) => +d.value)])
            .range([height, 0])

        const yAxis = svg.append('g').call(d3.axisLeft(y)).attr('id', 'axisRed')

        // Add a clipPath: everything out of this area won't be drawn.
        const clip = svg
            .append('defs')
            .append('clipPath')
            .attr('id', 'clip')
            .append('rect')
            .attr('width', width)
            .attr('height', height)
            .attr('x', 0)
            .attr('y', 0)

        // Add brushing
        const brush = d3
            .brushX() // Add the brush feature using the d3.brush function
            .extent([
                [0, 0],
                [width, height]
            ]) // initialise the brush area: start at 0,0 and finishes at width,height: it means I select the whole graph area
            .on('end', updateChart) // Each time the brush selection changes, trigger the 'updateChart' function

        // Create an area generator
        areaGenerator = d3
            .area()
            .x((d) => x(d.date))
            .y0(y(0))
            .y1((d) => y(d.value))

        const defs = svg.append('defs')

        const bgGradient = defs
            .append('linearGradient')
            .attr('id', 'bg-gradient')
            .attr('gradientTransform', 'rotate(90)')

        bgGradient.append('stop').attr('stop-color', ' #f5251b ').attr('offset', '20%')
        bgGradient.append('stop').attr('stop-color', '#34cc04').attr('offset', '100%')

        // Add the brushing
        const area = svg.append('g').attr('clip-path', 'url(#clip)')

        // Add the area
        area.append('path')
            .datum(newArray)
            .attr('class', 'myArea') // I add the class myArea to be able to modify it later on.
            .attr('fill', 'url(#bg-gradient)')
            .attr('fill-opacity', 0.7)
            .attr('stroke', 'none')
            .attr('stroke-width', 1)
            .attr('d', areaGenerator)

        area.append('g').attr('class', 'brush').call(brush)
        // A function that set idleTimeOut to null
        let idleTimeout
        function idled() {
            idleTimeout = null
        }

        // A function that update the chart for given boundaries
        function updateChart(event) {
            // What are the selected boundaries?
            let extent = event.selection

            // If no selection, back to initial coordinate. Otherwise, update X axis domain
            if (!extent) {
                if (!idleTimeout) return (idleTimeout = setTimeout(idled, 350)) // This allows to wait a little bit
                x.domain([4, 8])
            } else {
                date.value = [x.invert(extent[0]), x.invert(extent[1])]
                x.domain([x.invert(extent[0]), x.invert(extent[1])])
                area.select('.brush').call(brush.move, null) // This remove the grey brush area as soon as the selection has been done
            }

            // Update axis and area position
            xAxis.transition().duration(1000).call(d3.axisBottom(x))
            area.select('.myArea').transition().duration(1000).attr('d', areaGenerator)
        }

        // If user double click, reinitialize the chart
        svg.on('dblclick', function () {
            x.domain(d3.extent(newArray, (d) => d.date))
            xAxis.transition().call(d3.axisBottom(x))

            area.select('.myArea').transition().attr('d', areaGenerator)
        })
    })
}

onMounted(() => {
    date.value = [startDate, endDate]
    initGraph()
})
</script>

<style>
header {
    padding: 0;
    width: 110%;
    margin-left: -5%;
    position: relative;
}

#my_dataviz {
    background-color: rgb(255, 255, 255);
    border-radius: 20px;
    box-shadow: rgba(75, 75, 75, 0.24) 0px 3px 8px;
    width: max-content;
    margin-left: auto;
    margin-right: auto;
}
#datePicker {
    width: 250px;
}

#axisRed line {
    stroke: rgb(0, 0, 0);
}

#axisRed path {
    stroke: rgb(0, 0, 0);
}

#axisRed text {
    fill: rgb(0, 0, 0);
}
</style>
