<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";

	export const myName = "Minsuk Kahng";

	// Define global variables that will be used in HTML
	let data;
	let yScaleWithD3; // Note that I changed this variable's name from xScaleNew
	let yScaleTicks = [];
	// let colorScale; We will be doing this next week.

	let maximumPopulation = 9999999999; // better to be initialized to avoid "undefined" errors
	
	// Sizes of canvas and bars
	const barWidth = 15; // better to use variables
	const paddingBetweenBars = 5;
	const maximumHeight = 250;


	// Code that is executed once at the beginning
	onMount(async () => {
		const fetched = await fetch("/static/census2000.json");
		let rawData = (await fetched.json()).demographic_groups;
		console.log(rawData);

		// Filter data only to use 19 rows among 76 rows
		console.log("before filtered: ", rawData.length);
		data = rawData.filter((record) => {
			return record.year == 2000 && record.sex == "F";
		});
		console.log("after filtered: ", data.length);

		// Obtain the maximum population value
		let arrayOfOnlyPopulation = data.map((record) => {
			return record.people;
		})
		console.log(arrayOfOnlyPopulation);
		maximumPopulation = Math.max(...arrayOfOnlyPopulation);
		console.log(maximumPopulation);

		// Create a scale function for the population attribute using D3
		yScaleWithD3 = scaleLinear()
			.domain([0, maximumPopulation])
			.range([maximumHeight, 0]);

		// Create axis ticks
		yScaleTicks = yScaleWithD3.ticks(5, ","); // I want roughly 5 ticks.
		console.log(yScaleTicks);
	});

	// The first version of the scale function
	// It takes a population value as input and return the width of the bar
	// function yScaleOld(value) {
	// 	//return value / 50000;
	// 	return (value / maximumPopulation) * maximumHeight;
	// }

	// The first version of the scale function
	// We will be working on the color scale again next week.
	// function colorAge(value) {
	// 	if (value == "0-4" || value == "5-9") {
	// 		return "blue";
	// 	} else {
	// 		return "green";
	// 	}
	// }
</script>

<main>
	<h1>In-Class Acitivity by {myName}!</h1>

	<svg id="visualization">
		{#if data !== undefined}
			<!-- 50 pixel of space for putting y-axis on the left and 30 pixels at the top -->
			<g id="barChartArea"
				transform="translate(50, 30)">

				<!-- Bars. Note that this version is for a vertical bar chart. -->
				<g id="barChartBars">
					{#each data as record, index}
						<g transform="translate({index * (barWidth + paddingBetweenBars)}, 0)">
							<!-- Note that "height" is "maxHeight - yScale()" -->
							<rect class="bar"
								x="0"
								y={yScaleWithD3(record.people)}
								width={barWidth}
								height={maximumHeight - yScaleWithD3(record.people)} 
							/>
							<text class="barLabelBottom" 
								x={barWidth * 0.5} y={maximumHeight + 9}>
								{record.age}
							</text>
						</g>
					{/each}
				</g>

				<!-- x-axis on the bottom -->
				<g id="barChartXAxisArea">
					<line class="axisLine"
						x1="-5" y1={maximumHeight}
						x2={data.length * (barWidth + paddingBetweenBars)} y2={maximumHeight}
					/>
				</g>

				<!-- y-axis on the left side -->
				<g id="barChartYAxisArea"
					transform="translate(-5, 0)">
					<!-- y-axis for the population attribute -->
					{#each yScaleTicks as tick}
						<text x="-6" y={yScaleWithD3(tick) + 3}>{tick}</text>
						<line class="tick"
							x1="0"  y1={yScaleWithD3(tick)} 
							x2="-5" y2={yScaleWithD3(tick)} 
						/>
					{/each}
					<line class="axisLine"
						x1="0" y1={maximumHeight}
						x2="0" y2="0"
					/>
				</g>
			</g>
		{/if}
	</svg>
</main>

<style>
	h1 {
		font-size: 1.5rem;
	}
	svg#visualization {
		width: 600px;
		height: 500px;
	}
	text {
		font-size: 0.5rem;
	}
	rect.bar {
		fill: steelblue;
	}
	text.barLabelBottom {
		font-size: 0.4rem;
		text-anchor: middle;
	}
	#barChartYAxisArea text {
		text-anchor: end;
	}
	.axisLine, .tick {
		stroke: darkgray;
	}
</style>
