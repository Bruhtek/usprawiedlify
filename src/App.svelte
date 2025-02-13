<script lang="ts">
	import { onMount } from "svelte";
	import { Confetti } from "svelte-confetti";
	import { elasticOut } from "svelte/easing";

	const STANDARD_REASONS = [
		'choroby',
		'bólu głowy',
		'bólu zęba',
		'bólu brzucha',
		'bólu kostki',
		'złego samopoczucia',
		'przeziębienia',
		'ryzyka odnowienia kontuzji na zajęciach WF-u',
		'zaleceń lekarza',
		'wizyty u dentysty',
		'wizyty u lekarza',
		'wizyty u fizjoterapeuty',
		'wizyty u psychologa',
		'badania',
		'rezonansu magnetycznego',
		'badania usg',
		'problemów komunikacyjnych',
		'usterki autobusu',
		'usterki tramwaju',
		'prac drogowych na drodze do szkoły',
		'problemów z samochodem',
		'wypadku na drodze',
		'wykolejenia tramwaju',
		'egzaminu na prawo jazdy',
		'ważnego egzaminu',
		'konieczności wypełnienia obowiązków domowych',
		'konieczności pomocy rodzicom',
		'wyjazdu',
		'wyjazdu rodzinnego',
		'spraw prywatnych',
		'ważnych spraw rodzinnych',
		'konieczności wyrobienia dokumentów',
		'konieczności odebrania dokumentów',
		'konieczności odebrania rodzeństwa ze szkoły',
		'konieczności zaprowadzenia rodzeństwa do lekarza',
		'konieczności opieki nad młodszym rodzeństwem',
		'konieczności wizyty u weterynarza',
		'ważnego spotkania',
		'pozaszkolnych zawodów sportowych',
		'pozaszkolnego konkursu',
		'przygotowywania się do sprawdzianu',
		'przygotowywania się do kartkówki',
		'przygotowywania się do odpowiedzi',
		'przygotowywania się do egzaminu',
		'przygotowywania się do konkursu',
		'potrzeby nadrobienia zaległości',
		'szkolenia',
		'przemęczenia',
		'uczestnictwa w wolontariacie',
		'uczestnictwa w ważnym wydarzeniu',
		'pomocy w organizacji wydarzenia',
		'wizyty u fryzjera',
		'wizyty u barbera'
	]

	const FUNNY_REASONS = [
		'wczorajszej imprezy',
		'ostrego kaca',
		'pobytu na izbie wytrzeźwień',
		'chorego psa',
		'internetowego turnieju pokera',
		'braku pieniędzy na bilet na autobus',
		"zbyt długiego meczu w CS'a",
		'złej daty w kalendarzu',
		'przekrzywionej półosi w samochodzie',
		'wydziedziczenia (szukam domu)',
		'takiego, że inni nie idą',
		'braku chęci do interakcji społecznych',
		'deszczu',
		'niewychodzenia z domu - boje się, że coś mi sie stanie',
		'nieprzyjemnego kontaktu z agresywnym kibicem',
		'dzisiejszych derbów Cracovii (mieszkam na Prokocimiu)',
		'braku butów na zmianę (boje się dyrekcji)',
		'kota na torbie',
	]

	let noFunAllowed: boolean = $state(true);

	let REASONS = $derived(noFunAllowed ? STANDARD_REASONS : FUNNY_REASONS)

	function whoosh(node: HTMLElement, params: { delay?: number, duration?: number, easing?: (t: number) => number }) {
		const existingTransform = getComputedStyle(node).transform.replace('none', '');

		return {
			delay: params.delay || 0,
			duration: params.duration || 400,
			easing: params.easing || elasticOut,
			css: (t: number, u: number) => `transform: ${existingTransform} scale(${t * 0.2 + 0.8})`
		};
	}

	let completed: boolean = $state(false);
	let currentReason = $state(0)
	let interval: number;

	function roll() {
		let newReason = Math.floor(Math.random() * 10000) % REASONS.length;
		while(newReason === currentReason) {
			newReason = Math.floor(Math.random() * 10000) % REASONS.length;
		}
		currentReason = newReason;
	}

	let intervalTime = $state(700);

	onMount(() => {
		interval = setInterval(() => {
			roll();
		}, intervalTime);

		return () => {
			clearInterval(interval);
		}
	})

	async function startRolling() {
		let initialIntervalTime = 50;
		completed = false;

		while(initialIntervalTime < 400) {
			clearInterval(interval);
			interval = setInterval(roll, initialIntervalTime);
			await new Promise(resolve => setTimeout(resolve, 800/initialIntervalTime * initialIntervalTime));
			initialIntervalTime *= 1.5;
		}

		clearInterval(interval);
		completed = true;
	}

	function copy() {
		const toCopy = `Dzień dobry,\nuprzejmie proszę o usprawiedliwienie mojej nieobecności dnia XX.XX.2025 z powodu ${REASONS[currentReason]}.\n\nZ poważaniem,\nXYZ`

		navigator.clipboard.writeText(toCopy);
	}
</script>

<div>
	<h1>Usprawiedlify</h1>
	<p>
		Dzień dobry,<br />uprzejmie proszę o usprawiedliwienie mojej nieobecności dnia XX.XX.2025 z powodu
	</p>
	{#key currentReason}
		<div class="reason" in:whoosh={{duration: intervalTime}}>
			{REASONS[currentReason]}
			{#if completed}
				<div class="center">
					<Confetti
						amount={800}
						x={[-6, 6]}
						y={[-4, 4]}
						delay={[0, 1000]}
					/>
				</div>
			{/if}
		</div>
	{/key}
	<p>
		Z poważaniem <br>
		XYZ
	</p>

	<div class="buttons">
		<label>
			Na poważnie?
			<input type="checkbox" bind:checked={noFunAllowed} />
		</label>
		<br />
		<br />
		<button onclick={startRolling}>
			Losuj!
		</button>

		{#if completed}
			<button class="copy-btn" onclick={copy}>
				Skopiuj do schowka
			</button>
		{/if}
	</div>
</div>

<style>
	p {
		font-size: 1.5rem;
	}
	.reason {
		font-size: 1.7rem;
		font-weight: bold;
		position: relative;
	}
	.center {
		position: absolute;
		top: 0;
		left: 50%;
		transform: translateX(-50%);
	}

	.copy-btn {
		position: absolute;
		top: 100px;
		left: 50%;
		transform: translateX(-50%);
	}
	.buttons {
		position: relative;
	}
</style>