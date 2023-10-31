
<template>
    <svg></svg> 
</template>

<script setup>
    import { onMounted } from 'vue';
    import * as d3 from 'd3'

    // import data1 from '../daily_prod_data.json'
    import data1 from '../test.json'
    import moment from 'moment';

    let oneArray = data1["SYSI-0020"];
    let newArray = []

    const width = 1928;
    const height = 500;
    const marginTop = 30;
    const marginRight = 0;
    const marginBottom = 30;
    const marginLeft = 140;

    function toInt(){
        for ( let i = 0; i < oneArray.length; i++){
            newArray.push({
                timestamp: moment(oneArray[i].timestamp),
                prod_kwh: oneArray[i].prod_kwh
            })    
        }
    }
    
    function initGraph(){

        // Declare the x (horizontal position) scale.
        let x = d3.scaleBand()
            .domain(newArray.map(data => data.timestamp))
            .range([marginLeft, width - marginRight])
            .padding(0.1);
    
        // Declare the y (vertical position) scale.
        let y = d3.scaleLinear()
            .domain([0, d3.max(newArray, (d) => d.prod_kwh)])
            .range([height - marginBottom, marginTop]);
    
        // Create the SVG container.
        let svg = d3.select("svg")
            .attr("width", width)
            .attr("height", height)
            .attr("viewBox", [0, 0, width, height])
            .attr("style", "max-width: 100%; height: auto;");

        // Add a rect for each bar.
        svg.append("g")
            .attr("fill", "steelblue")
        .selectAll()
        .data(newArray)
        .join("rect")
            .attr("x", (d) => x(d.timestamp))
            .attr("y", (d) => y(d.prod_kwh))
            .attr("height", (d) => y(0) - y(d.prod_kwh))
            .attr("width", x.bandwidth());

        // Add the x-axis and label.
        svg.append("g")
            .attr("transform", `translate(0,${height - marginBottom})`)
            .call(d3.axisBottom(x).tickFormat(d3.utcFormat("%B %d, %Y")))

        // Add the y-axis and label, and remove the domain line.
        svg.append("g")
            .attr("transform", `translate(${marginLeft},0)`)
            .call(d3.axisLeft(y).tickFormat((y) => (y).toFixed()))
            .call(g => g.select(".domain").remove())
            .call(g => g.append("text")
                .attr("x", -marginLeft)
                .attr("y", 15)
                .attr("fill", "currentColor")
                .attr("text-anchor", "start")
                .text("Production (kwh)"));

        // Return the SVG element.
        return svg.node();
    }

    onMounted(() => {

        toInt()
        initGraph()

    })



</script>