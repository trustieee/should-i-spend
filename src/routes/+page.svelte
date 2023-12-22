<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { Chart, registerables } from 'chart.js';
	Chart.register(...registerables);
	let amount = '';
	let years: number = 30;
	let increments = 5;
	let n = 1; // Compounding once per year
	let rates = [0.03, 0.05, 0.08]; // Interest rates of 3%, 5%, and 8%
	let tableData: { years: number; rate: number; value: number }[] = [];
	let canvas: HTMLCanvasElement;
	let chart: Chart<'line', number[], string>;
	let colors = ['#FF6347', '#40E0D0', '#ADFF2F']; // Tomato, Turquoise, GreenYellow

	// Function to calculate future value
	function calculateFutureValue(pv: number, r: number, n: number, t: number) {
		let fv = pv * Math.pow(1 + r / n, n * t);
		return fv;
	}

	$: {
		// Generate chart labels and datasets
		let labels = Array.from({ length: years / increments + 1 }, (_, i) => i * increments);
		let datasets = rates.map((r, index) => {
			let data = tableData.filter((d) => d.rate === r).map((d) => d.value);
			return {
				label: `Interest Rate: ${r * 100}%`,
				data: data,
				fill: false,
				borderColor: colors[index % colors.length] // Use a different color for each line
			};
		});

		// Update or create chart
		if (chart) {
			chart.data = { labels: labels.map(String), datasets };
			chart.update();
		} else if (canvas) {
			chart = new Chart(canvas.getContext('2d'), {
				type: 'line',
				data: { labels, datasets },
				options: {
					responsive: true
					// ... rest of your options ...
				}
			});
		}
	}

	onMount(() => {
		// Generate table data
		for (let t = 0; t <= years; t += increments) {
			for (let r of rates) {
				let value = calculateFutureValue(15, r, n, t); // Use 15 as the principal value
				tableData.push({ years: t, rate: r, value: value });
			}
		}

		// Generate chart labels and datasets
		let labels = Array.from({ length: years / increments + 1 }, (_, i) => i * increments);
		let datasets = rates.map((r, index) => {
			let data = tableData.filter((d) => d.rate === r).map((d) => d.value);
			return {
				label: `Interest Rate: ${r * 100}%`,
				data: data,
				fill: false,
				borderColor: colors[index % colors.length] // Use a different color for each line
			};
		});

		// Create chart
		chart = new Chart(canvas.getContext('2d'), {
			type: 'line',
			data: { labels, datasets },
			options: {
				responsive: true,
				title: {
					display: true,
					text: 'Future Value of $1'
				},
				scales: {
					y: {
						beginAtZero: true,
						ticks: {
							// Ensure that ticks are generated for small values
							precision: 0.01
						}
					}
				}
			}
		});
	});

	onDestroy(() => {
		if (chart) {
			chart.destroy();
		}
	});
	function selectOnFocus(node: HTMLInputElement) {
		node.addEventListener('focus', () => {
			node.select();
		});

		return {
			destroy() {
				node.removeEventListener('focus', () => {
					node.select();
				});
			}
		};
	}

	$: {
		tableData = []; // Clear the table data
		for (let rate of rates) {
			for (let i = 0; i <= years; i += increments) {
				tableData.push({
					years: i,
					rate: rate,
					value: calculateFutureValue(Number(amount), rate, n, i)
				});
			}
		}
	}
</script>

<div class="dark:bg-gray-800 min-h-screen flex flex-col justify-start pt-16">
	<div class="relative py-3 sm:mx-auto overflow-x-auto px-5">
		<h3 class="italic text-slate-500">should you spend...?</h3>
		<!-- <h1 class="text-9xl mb-16">${Number(amount).toFixed(2)}</h1> -->
		<div class="relative flex items-center w-full">
			<span class="absolute left-0 transform -translate-y-1/2 ml-8 text-6xl">$</span>
			<!-- svelte-ignore a11y-autofocus -->
			<input
				autofocus
				bind:value={amount}
				use:selectOnFocus
				placeholder="Enter an amount"
				class="text-9xl ml-3 mb-16 w-full bg-transparent border-none outline-none placeholder-gray-700 pl-16"
				style="width: 90%;"
				on:keydown={(event) => {
					// Allow: Ctrl+A, Ctrl+C, Ctrl+V, Ctrl+X
					if (
						(event.ctrlKey === true || event.metaKey === true) &&
						['a', 'c', 'v', 'x'].includes(event.key.toLowerCase())
					) {
						return;
					}
					if (
						!['Backspace', 'Delete', 'Tab', 'Escape', 'Enter', '.', 'Delete'].includes(event.key) &&
						(event.key < '0' || event.key > '9')
					) {
						event.preventDefault();
					}
					// prevent '.' if already exists in the value
					const currentValue = event.currentTarget.value;
					if (event.key === '.' && currentValue.indexOf('.') !== -1) {
						event.preventDefault();
					}
					// limit to two decimal places
					if (event.key === '.' || (event.key >= '0' && event.key <= '9')) {
						const parts = currentValue.split('.');
						if (parts.length === 2 && parts[1].length >= 2) {
							event.preventDefault();
						}
					}
				}}
			/>
		</div>
		<div class="flex justify-around space-x-24">
			{#each rates as rate (rate)}
				<div>
					<h1 class="ml-6">Compound rate {rate * 100}%</h1>
					<table class="table table-lg w-full mb-16" style="min-width: 500px;">
						<thead>
							<tr>
								<th>Years</th>
								<th>Result</th>
							</tr>
						</thead>
						<tbody>
							{#if +amount === 0}
								<tr>
									<td colspan="2" class="text-center italic text-gray-500"
										>Enter an amount to see the results</td
									>
								</tr>
							{:else}
								{#each tableData.filter((row) => row.rate === rate) as row (row.years)}
									<tr class="hover:bg-slate-700 transition-colors duration-200">
										<td>{row.years}</td>
										<td>${row.value.toFixed(2)}</td>
									</tr>
								{/each}
							{/if}
						</tbody>
					</table>
				</div>
			{/each}
		</div>
		<div class="flex justify-center">
			<canvas bind:this={canvas}></canvas>
		</div>
		<p class="text-slate-600 mb-16 mt-16">
			A = P(1 + [{rates.map((rate) => rate * 100 + '%').join(', ')}])/1)^((1)[0-30])
		</p>
	</div>
	<!-- {/if} -->
</div>
