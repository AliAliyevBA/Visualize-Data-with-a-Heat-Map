# Visualize-Data-with-a-Heat-Map
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Heat Map</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
        }

        svg {
            display: block;
            margin: 0 auto;
        }

        .cell {
            stroke: #555;
        }

        #tooltip {
            position: absolute;
            background-color: lightgray;
            padding: 5px;
            border-radius: 5px;
            display: none;
            font-size: 14px;
        }

        #legend rect {
            stroke: black;
        }
    </style>
</head>
<body>
    <h1 id="title">Global Temperature Heat Map</h1>
    <p id="description">1754 - 2015: Global Land-Surface Temperature Variations</p>
    
    <svg id="chart"></svg>
    <div id="tooltip"></div>

    <!-- Load the FCC Test Script -->
    <script src="https://cdn.freecodecamp.org/testable-projects-fcc/v1/bundle.js"></script>

    <script>
        const url = "https://raw.githubusercontent.com/freeCodeCamp/ProjectReferenceData/master/global-temperature.json";

        // Set dimensions
        const margin = { top: 100, right: 30, bottom: 100, left: 100 };
        const width = 1000 - margin.left - margin.right;
        const height = 400 - margin.top - margin.bottom;

        // Create SVG
        const svg = d3.select("#chart")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
          .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);

        // Tooltip
        const tooltip = d3.select("#tooltip");

        d3.json(url).then(data => {
            const baseTemp = data.baseTemperature;
            const monthlyData = data.monthlyVariance;

            // Parse data
            monthlyData.forEach(d => {
                d.month -= 1; // Adjust month to zero-indexed
                d.year = +d.year;
                d.variance = +d.variance;
                d.temp = baseTemp + d.variance;
            });

            // Define scales
            const xScale = d3.scaleBand()
                .domain(monthlyData.map(d => d.year))
                .range([0, width])
                .padding(0.05);

            const yScale = d3.scaleBand()
                .domain([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11])
                .range([0, height])
                .padding(0.05);

            const colorScale = d3.scaleQuantile()
                .domain(d3.extent(monthlyData, d => d.temp))
                .range(["#4575b4", "#91bfdb", "#fee090", "#fc8d59", "#d73027"]);

            // Define axes
            const xAxis = d3.axisBottom(xScale).tickValues(xScale.domain().filter(year => year % 10 === 0));
            const yAxis = d3.axisLeft(yScale).tickFormat(month => d3.timeFormat("%B")(new Date(0, month)));

            // Append axes
            svg.append("g")
                .attr("id", "x-axis")
                .attr("transform", `translate(0, ${height})`)
                .call(xAxis);

            svg.append("g")
                .attr("id", "y-axis")
                .call(yAxis);

            // Append cells
            svg.selectAll(".cell")
                .data(monthlyData)
                .enter()
                .append("rect")
                .attr("class", "cell")
                .attr("x", d => xScale(d.year))
                .attr("y", d => yScale(d.month))
                .attr("width", xScale.bandwidth())
                .attr("height", yScale.bandwidth())
                .attr("fill", d => colorScale(d.temp))
                .attr("data-month", d => d.month)
                .attr("data-year", d => d.year)
                .attr("data-temp", d => d.temp)
                .on("mouseover", (event, d) => {
                    tooltip.style("display", "block")
                        .style("left", `${event.pageX + 5}px`)
                        .style("top", `${event.pageY - 28}px`)
                        .attr("data-year", d.year)
                        .html(`Year: ${d.year}<br>Month: ${d3.timeFormat("%B")(new Date(0, d.month))}<br>Temp: ${d.temp.toFixed(2)}°C`);
                })
                .on("mouseout", () => tooltip.style("display", "none"));

            // Append legend
            const legendWidth = 300;
            const legendHeight = 20;
            const legend = svg.append("g")
                .attr("id", "legend")
                .attr("transform", `translate(${(width - legendWidth) / 2}, ${height + margin.bottom / 2})`);

            const legendScale = d3.scaleLinear()
                .domain(d3.extent(monthlyData, d => d.temp))
                .range([0, legendWidth]);

            const legendAxis = d3.axisBottom(legendScale)
                .tickSize(legendHeight)
                .tickFormat(d3.format(".1f"))
                .ticks(5);

            const legendCells = legend.selectAll("rect")
                .data(colorScale.range())
                .enter()
                .append("rect")
                .attr("x", (d, i) => i * (legendWidth / colorScale.range().length))
                .attr("y", 0)
                .attr("width", legendWidth / colorScale.range().length)
                .attr("height", legendHeight)
                .attr("fill", d => d);

            legend.append("g")
                .attr("transform", `translate(0, ${legendHeight})`)
                .call(legendAxis);
        });
    </script>
</body>
</html>
