<script>
	import { onMount, onDestroy } from 'svelte';

	import { countdownOver } from '$lib/store/app-stores';

	let countdown;
	let secondsRemaining = 30;

	function updateCountdown() {
		if (secondsRemaining <= 0) {
			// Countdown has ended
			clearInterval(countdown);
			countdownOver.set(true);
			console.log('Countdown has ended!');
		} else {
			// Display the remaining seconds
			console.log(`${secondsRemaining}s`);
			secondsRemaining--;

			// Randomly increase the time by 1
			if (Math.random() < 0.1) {
				secondsRemaining++;
			}
		}
	}
	onMount(() => {
		// Update the countdown initially
		updateCountdown();
		countdownOver.set(false);

		// Set up an interval to update the countdown every second
		countdown = setInterval(updateCountdown, 1000);
	});

	onDestroy(() => {
		// Clear the interval when the component is destroyed
		clearInterval(countdown);
		countdownOver.set(false);
	});
</script>

<!-- Add your HTML here -->
<div>
	<p>Estimated Time Left: {secondsRemaining}s</p>
</div>
