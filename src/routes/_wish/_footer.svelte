<script>
	import { getContext, setContext } from 'svelte';
	import { t } from 'svelte-i18n';
	import hotkeys from 'hotkeys-js';
	import { fly } from 'svelte/transition';
	import {
		acquaint,
		assets,
		mobileMode,
		stardust,
		starglitter,
		intertwined,
		wishAmount,
		activeVersion,
		multipull,
		editorMode,
		preloadVersion,
		editID,
		genesis,
		primogem,
		infoPannelPaimon
	} from '$lib/store/app-stores';
	import { playSfx } from '$lib/helpers/audio/audio';
	import { isNewOutfitReleased } from '$lib/helpers/outfit';
	import { localBanner } from '$lib/helpers/custom-banner';
	import { pushToast } from '$lib/helpers/toast';

	import Icon from '$lib/components/Icon.svelte';
	import NoticeMark from '$lib/components/NoticeMark.svelte';
	import ButtonGeneral from '$lib/components/ButtonGeneral.svelte';
	import EpitomizedButton from './epitomized-path/_button.svelte';
	import BannerPublisher from '../_custom-banner/Publisher.svelte';
	import InfoPannelP from './_info-pannel-p.svelte';

	export let bannerType = 'beginner';

	$: isBeginner = bannerType === 'beginner';
	$: isEvent = bannerType.match('event');
	$: currencyUsed = isEvent ? $intertwined : $acquaint;
	$: fateType = isEvent ? 'intertwined' : 'acquaint';
	$: isUnlimited = $wishAmount === 'unlimited';

	// Notice Mark
	$: patch = $activeVersion.patch;
	$: openedNotices = [`outfits-${patch}`, `recomended-${patch}`];
	$: hasNewOutfit = isNewOutfitReleased(patch);

	const onWish = getContext('onWish');
	const readyToPull = getContext('readyToPull');

	const navigate = getContext('navigate');
	const changePage = (page) => {
		navigate(page);
		if (['inventory', 'history'].includes(page)) return playSfx(page);
		if (page === 'shop') return playSfx('shopopen');
		return playSfx();
	};

	let bubbleMsgs = ["Don't forget! You can always get your wishes for free by playing the game."];
	let showBubble = false;
	setInterval(() => {
		if (Math.random() < 0.1) {
			showBubble = true;
			setTimeout(() => {
				showBubble = false;
			}, 5000);
		}
	}, 6000);

	let showElement = false;

	const roll = getContext('doRoll');
	const handleSingleRollClick = () => {
		playSfx('roll');
		roll(1, bannerType);
	};
	const handleMultiRollClick = () => {
		playSfx('roll');
		roll(isBeginner ? 10 : $multipull, bannerType);
	};

	// Footer for Editor
	const finishAndWish = async () => {
		playSfx();
		const isComplete = await localBanner.isComplete($editID);
		if (isComplete) return preloadVersion.set({ patch: 'Custom', phase: $editID });

		// Benner not Complete
		const toastMsg = $t('customBanner.completeAllField');
		pushToast({ message: toastMsg, type: 'error' });
		return;
	};

	let showUploader = false;
	const publishBanner = async () => {
		playSfx();
		const isComplete = await localBanner.isComplete($editID);
		if (isComplete) return (showUploader = true);
		const toastMsg = $t('customBanner.completeAllField');
		pushToast({ message: toastMsg, type: 'error' });
	};

	setContext('closePublisher', () => (showUploader = false));
	setContext('publishDone', () => {
		preloadVersion.set({ patch: 'Custom', phase: $editID });
		showUploader = false;
		playSfx('close');
	});

	setContext('publishError', () => {
		const toastMsg = $t('customBanner.networkError');
		pushToast({ message: toastMsg, type: 'error' });
		showUploader = false;
	});

	// ShortCut
	const appReady = getContext('appReady');
	hotkeys('enter', 'index', (e) => {
		if (!$appReady || $onWish || $editorMode) return;
		e.preventDefault();
		handleMultiRollClick();
	});

	hotkeys('shift+enter', 'index', (e) => {
		if (!$appReady || $onWish || isBeginner || $editorMode) return;
		e.preventDefault();
		handleSingleRollClick();
	});

	hotkeys('s,c,h,d', 'index', (e) => {
		if (!$appReady || $onWish || $editorMode) return;
		e.preventDefault();
		const [key] = hotkeys.getPressedKeyString();
		const to = key.toLocaleLowerCase();
		if (to === 's') return changePage('shop');
		if (to === 'c') return changePage('inventory');
		if (to === 'h') return changePage('history');
		if (to === 'd') return changePage('details');
	});
</script>

<!-- {#if displayPopup}
	<div class="overlay-div">
		<div class="white-rectangle">
			<p style="text-align: center; font-size: 1.5rem; margin-top: 10px;">
				Hey buddy, I see you're pulling in this banner.
			</p>
			<div class="fate-conversion">
				1 <Icon type="intertwined" style="transform: translateY(-3px); margin: 0 5px;" /> =
				{price.toFixed(2)}€
			</div>
			<div class="could-buy">
				<p style="text-align: center; font-size: 1.5rem; margin-top: 10px;">
					You could buy {Math.floor($primogem / 160)}
					<Icon type="intertwined" /> with your current Starglitter balance.
				</p>
			</div>
			<div class="bottom-left">
				<div class="bottom-left-content">
					<p style="text-align: center; font-size: 1.5rem; margin-top: 10px;">
						You could buy {Math.floor($primogem / 160)}
						<Icon type="intertwined" /> with your current Starglitter balance.
					</p>
				</div>
			</div>
		</div>

		<style>
			.bottom-left {
				position: relative;
				width: 100%;
				height: 100%;
				border: 5px solid red;
			}

			.white-rectangle {
				/* 70% of screen width */
				width: 70vw;
				height: 70vh;
				background-color: white;
				position: absolute;
				top: 50%;
				left: 50%;
				transform: translate(-50%, -50%);
				/* rounded corners */
				border-radius: 25px;
				/* add shadow on the bottom */
				box-shadow: 0px 0px 20px 0px rgba(0, 0, 0, 0.1);
				/* a bit of transparency */
				opacity: 0.9;
			}
			.fate-conversion {
				/* center horizontally */
				display: flex;
				justify-content: center;
			}
		</style>
	</div>
{/if} -->

{#if showUploader}
	<BannerPublisher />
{/if}

<div id="footer" style="width: 100%; height: 100%">
	{#if !$editorMode}
		<div class="footer-info">
			{#if !$mobileMode}
				{#if bannerType === 'weapon-event'}
					<EpitomizedButton />
				{/if}
				<div class="wish">
					<div class="starglitter">
						<Icon type="starglitter" />
						<span> {$starglitter} </span>
					</div>
					<div class="stardust">
						<Icon type="stardust" />
						<span> {$stardust} </span>
					</div>
				</div>
			{/if}
		</div>
	{/if}

	<div class="row" style="--bg:url({$assets['button.webp']})">
		{#if !$editorMode}
			<div class="left menu-button">
				<ButtonGeneral on:click={() => changePage('shop')}>
					{#if hasNewOutfit}
						<NoticeMark name={openedNotices} style="transform: translateX(70%) translateY(-80%)" />
					{/if}
					{$t('shop.text')}
				</ButtonGeneral>

				<ButtonGeneral on:click={() => changePage('inventory')}>
					{$t('inventory.text')}
				</ButtonGeneral>
				<ButtonGeneral on:click={() => changePage('history')}>
					{$t('history.text')}
				</ButtonGeneral>
			</div>
			<div>
				{#if showBubble}
					<div class="bubble" transition:fly={{ y: -100, duration: 1000 }}>
						{bubbleMsgs[Math.floor(Math.random() * bubbleMsgs.length)]}
					</div>
				{/if}
				<img
					src="/images/image.png"
					alt="Paimon"
					style="width: 64px; height: 64px; position: absolute; left: 50%; transform: translateX(-50%); bottom: 0; margin-bottom: 10px; z-index: 100;"
					on:click={() => (infoPannelPaimon.update((v) => !v), playSfx('click'))}
				/>
			</div>
			{#if $infoPannelPaimon}
				<InfoPannelP />
			{/if}

			<div class="right roll-button">
				{#if !isBeginner}
					<button
						class="single wish-button"
						on:click={handleSingleRollClick}
						disabled={$onWish || !$readyToPull}
					>
						<div class="top">{$t('wish.rollButton', { values: { count: '×1' } })}</div>
						<div class="bottom">
							<Icon type={fateType} />
							<span style="margin-left: 7px" class:red={currencyUsed < 1 && !isUnlimited}>
								x 1
							</span>
							<span style="color: black; margin-left: 7px"> (3.17€) </span>
						</div>
					</button>
				{/if}

				<button
					class="ten wish-button"
					on:click={handleMultiRollClick}
					disabled={$onWish || !$readyToPull}
				>
					{#if bannerType === 'beginner'}
						<span class="discount">-20%</span>
					{/if}

					<div class="top">
						{$t('wish.rollButton', { values: { count: `×${isBeginner ? 10 : $multipull}` } })}
					</div>

					<div class="bottom">
						<Icon type={fateType} />
						{#if isBeginner}
							<span style="margin-left: 7px" class:red={currencyUsed < 8 && !isUnlimited}>
								x 8
							</span>
						{:else}
							<span style="margin-left: 7px" class:red={currencyUsed < $multipull && !isUnlimited}>
								x {$multipull}
							</span>
							<span
								style="color: black; margin-left: 7px"
								class:red={currencyUsed < 1 && !isUnlimited}
							>
								(31.70€)
							</span>
						{/if}
					</div>
				</button>
			</div>
		{:else}
			<div class="left menu-button" />
			<div class="right roll-button">
				<button
					class="wish-button"
					style="flex-direction: row; line-height: 0;"
					on:click={finishAndWish}
				>
					<i class="gi-primo-star" style="transform: translateX(-50%);" />
					<span> {$t('customBanner.finishAndWish')} </span>
				</button>

				<button
					class="wish-button"
					style="flex-direction: row; line-height: 0;"
					on:click={publishBanner}
				>
					<i class="gi-share" style="transform: translateX(-50%);" />
					{#await localBanner.isHostedBanner($editID)}
						<span> {$t('customBanner.publish')} </span>
					{:then isHosted}
						{#if isHosted}
							<span> {$t('customBanner.updateAndShare')} </span>
						{:else}
							<span> {$t('customBanner.publish')} </span>
						{/if}
					{/await}
				</button>
			</div>
		{/if}
	</div>
</div>

<style>
	.bubble {
		z-index: 100;
		position: fixed;
		bottom: 90px;
		left: 50%;
		transform: translateX(-50%);

		background: #f9f9f9;
		border-radius: 0.4em;
		width: fit-content;
		padding: 10px;
		margin: 0 auto;
	}
	.bubble:after {
		content: '';
		position: absolute;
		bottom: 0;
		left: 50%;
		width: 0;
		height: 0;
		border: 20px solid transparent;
		border-top-color: #f9f9f9;
		border-bottom: 0;
		border-left: 0;
		margin-left: 10px;
		margin-bottom: -20px;
	}

	#footer {
		position: relative;
	}

	.red {
		color: #de2f22 !important;
	}

	.footer-info {
		position: absolute;
		left: 5%;
		bottom: 75%;
		align-items: center;
		display: flex;
		flex-direction: column;
		pointer-events: none;
	}

	.wish > div {
		display: inline-flex;
		align-items: center;
		margin-right: 5px;
		padding: 2px 20px 2px 2px;
		font-size: 0.9rem;
	}
	.wish span {
		margin-left: 10px;
		color: #fff;
		text-shadow: 0 0 3px rgba(0, 0, 0, 0.5);
	}

	.wish-button {
		transform: scale(1);
		transition: all 0.2s;
		color: #4a5265;
		text-decoration: none;
		z-index: +5;
		position: relative;
	}

	.wish-button:active {
		filter: brightness(85%);
	}
	button:active:not(:disabled) {
		transform: scale(0.95);
	}

	.row {
		width: 100%;
		height: 100%;
		padding: 0 5%;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.roll-button {
		text-align: right;
	}

	.roll-button button {
		background-image: var(--bg);
		background-size: contain;
		background-position: center;
		background-repeat: no-repeat;
		width: 230px;
		aspect-ratio: 355/88;
		margin: 0 5px;
		display: inline-flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
		color: #a49a90;
		transition: all 0.2s;
	}

	.roll-button button:disabled {
		filter: brightness(0.5);
	}

	.roll-button button .bottom {
		display: flex;
		align-items: center;
	}

	.discount {
		background-color: #8ab958;
		position: absolute;
		z-index: +2;
		left: 15%;
		top: -5%;
		border-radius: 20px;
		color: #fff;
		transform: scale(0.8) translateX(-50%);
		padding: 0.2rem 0.5rem;
	}

	@media screen and (min-width: 750px) {
		.discount {
			font-size: 0.7rem;
		}
		.roll-button {
			white-space: nowrap;
		}
	}

	/* mobile */
	:global(.mobile) .row {
		padding: 1rem 0;
		align-items: flex-end;
	}

	:global(.mobile .menu-button) {
		padding-left: 1%;
		white-space: nowrap;
	}
	:global(.mobile) .roll-button {
		margin-right: 40px !important;
		white-space: nowrap;
	}
	:global(.mobile) .roll-button button {
		font-size: 0.75rem;
	}
	:global(.mobile) .roll-button :global(img) {
		transform: scale(0.7);
	}
	:global(.mobile) .bottom {
		margin-top: -3px;
	}

	/* Media Query */

	@media screen and (max-width: 925px) {
		button {
			padding: 2px 15px;
			margin: 2px 5px;
		}

		.roll-button button {
			width: 175px;
			margin: 0;
		}
		.roll-button :global(img) {
			transform: scale(0.8);
		}
	}

	@media screen and (max-width: 700px) {
		.roll-button {
			width: 100%;
		}
	}
	@media screen and (max-width: 550px) {
		.menu-button {
			width: 100%;
		}
		.roll-button {
			width: auto;
		}
	}

	@media screen and (max-width: 400px) {
		.roll-button button {
			width: 140px;
			margin: 0;
		}
		.roll-button :global(img) {
			transform: scale(0.7);
		}
		.roll-button .bottom {
			margin-top: -3px;
		}
	}
</style>
