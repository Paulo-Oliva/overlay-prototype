<script>
	import { infoPannelPaimon, buyReasons, moneySpentOnTheG } from '$lib/store/app-stores';
	import { onMount } from 'svelte';
	import Chart from 'chart.js/auto';

	onMount(() => {
		const barChartData = {
			labels: ['October', 'November', 'December', 'January'],
			datasets: [
				{
					label: 'Money spent on wishes (EUR)',
					backgroundColor: 'rgba(255, 99, 132, 0.5)',
					borderColor: 'rgba(255, 99, 132, 1)',
					borderWidth: 1,
					data: [14.99, 36.99, 23.99, $moneySpentOnTheG]
				}
			]
		};

		// Create bar graph
		const barGraphCtx = document.getElementById('barGraph').getContext('2d');
		new Chart(barGraphCtx, {
			type: 'bar',
			data: barChartData
		});
	});

	function getOverallSat() {
		// In a list of emotions, find the most common one
		let happyCount = 0;
		let neutralCount = 0;
		let sadCount = 0;
		for (let i = 0; i < $buyReasons.length; i++) {
			if ($buyReasons[i].emotion === 'happy') {
				happyCount++;
			} else if ($buyReasons[i].emotion === 'neutral') {
				neutralCount++;
			} else if ($buyReasons[i].emotion === 'sad') {
				sadCount++;
			}
		}
		if (happyCount > neutralCount && happyCount > sadCount) {
			return 'Happy';
		} else if (neutralCount > happyCount && neutralCount > sadCount) {
			return 'Neutral';
		} else if (sadCount > happyCount && sadCount > neutralCount) {
			return 'Sad';
		} else if (happyCount === neutralCount && happyCount > sadCount) {
			return 'Happy';
		} else if (happyCount === sadCount && happyCount > neutralCount) {
			return 'Neutral';
		} else if (neutralCount === sadCount && neutralCount > happyCount) {
			return 'Neutral';
		} else {
			return 'Neutral';
		}
	}

	let avgSat = getOverallSat();
</script>

<div class="popup">
	<div class="content">
		<aside>
			<div>
				Total Spent:
				<br />
				<b>{$moneySpentOnTheG}€</b>
			</div>
			<div>
				Overall Satisfaction:
				<br />
				<b>{avgSat}</b>
			</div>
		</aside>
		<h1>Info Pannel</h1>
		<div class="panel">
			<h2>Money Spent</h2>
			<!-- Bar Graph -->
			<canvas id="barGraph" />
		</div>
		<div class="recent">
			<h2>Recent Purchases</h2>
			{#each $buyReasons as r}
				<div class="panel">
					<div style="display: flex; justify-content: space-between;">
						<div class="reason">
							<p>{r.date} - {r.cost}€</p>
							<p>{r.reason}</p>
						</div>
						<div class="emotions">
							<p>How did this make you feel?</p>
							<!-- Happy/Neutral/Sad Selectable Icons -->
							<div style="display: flex; justify-content: space-between;">
								<div style="display: flex; flex-direction: column; align-items: center;">
									{#if r.emotion === 'happy'}
										<img
											style="border: 2px solid black; border-radius: 5px;"
											src="/images/happy.png"
											alt="Happy"
											width="50"
											on:click={() => {
												r.emotion = 'happy';
												avgSat = getOverallSat();
											}}
										/>
									{:else}
										<img
											src="/images/happy.png"
											alt="Happy"
											width="50"
											on:click={() => {
												r.emotion = 'happy';
												avgSat = getOverallSat();
											}}
										/>
									{/if}
								</div>
								<div style="display: flex; flex-direction: column; align-items: center;">
									{#if r.emotion === 'neutral'}
										<img
											style="border: 2px solid black; border-radius: 5px;"
											src="/images/neutral.png"
											alt="Neutral"
											width="50"
											on:click={() => {
												r.emotion = 'neutral';
												avgSat = getOverallSat();
											}}
										/>
									{:else}
										<img
											src="/images/neutral.png"
											alt="Neutral"
											width="50"
											on:click={() => {
												r.emotion = 'neutral';
												avgSat = getOverallSat();
											}}
										/>
									{/if}
								</div>
								<div style="display: flex; flex-direction: column; align-items: center;">
									{#if r.emotion === 'sad'}
										<img
											style="border: 2px solid black; border-radius: 5px;"
											src="/images/sad.png"
											alt="Sad"
											width="50"
											on:click={() => {
												r.emotion = 'sad';
												avgSat = getOverallSat();
											}}
										/>
									{:else}
										<img
											src="/images/sad.png"
											alt="Sad"
											width="50"
											on:click={() => {
												r.emotion = 'sad';
												avgSat = getOverallSat();
											}}
										/>
									{/if}
								</div>
							</div>
						</div>
					</div>
				</div>
			{/each}
		</div>

		<!-- <button class="close-button" on:click={() => infoPannelPaimon.set(false)}>Close</button> -->
		<div>
			<p style="font-size: 0.8rem; margin-top: 10px;">
				Note: We want to show you how much money you are spending on the game.
			</p>
			<p style="font-size: 0.8rem;">
				We are not saying that you should not spend money on the game, but we want you to be aware
				of how much you are spending.
			</p>
		</div>
	</div>
</div>

<style>
	.reason {
		width: 550px;
	}

	aside div {
		margin-bottom: 30px;
		margin-top: 30px;
	}

	/* Add overlay to emotion images */
	.content .recent .panel img:hover {
		border: 2px solid gray;
		border-radius: 5px;
	}

	/* Add a selection state to emotion images */
	.content .recent .panel img:active {
		border: 2px solid black;
		border-radius: 5px;
	}

	.popup {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.5);
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.content > aside {
		position: absolute;
		top: 35%;
		left: 12%;
		width: 15%;
		text-align: center;
		border: 1px solid #ddd;
		border-radius: 5px;
	}

	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		background-color: white;
		padding: 20px;
		border-radius: 5px;
		width: 80%;
		height: 78%;
		/* overflow: auto; */
		overflow-y: scroll;
	}

	/* .close-button {
		margin-top: 10px;
		padding: 5px 10px;
		background-color: #ff0000;
		color: white;
		border: none;
		border-radius: 5px;
		cursor: pointer;
	} */

	.panel {
		height: auto;
		width: 600px;
		border: 1px solid #ddd;
		padding: 20px;
		margin: 20px;
		border-radius: 5px;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
	}

	canvas {
		max-width: 100%;
		height: auto;
	}
</style>
