<script lang="ts">
	let amount = '';
	let years = 30;
	let increments = 1;
	let n = 1; // Compounding once per year
	let rates = [0.03, 0.05, 0.08]; // Interest rates of 3%, 5%, and 8%
	let tableData: { years: number; rate: number; value: number }[] = [];

	// Function to calculate future value
	function calculateFutureValue(pv: number, r: number, n: number, t: number) {
		return pv * Math.pow(1 + r / n, n * t);
	}

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
			<input
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
							{#if amount == 0}
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
		<div>chart here</div>
		<p class="text-slate-600 mb-16 mt-16">
			A = P(1 + [{rates.map((rate) => rate * 100 + '%').join(', ')}])/1)^((1)[0-30])
		</p>
	</div>
	<!-- {/if} -->
</div>
