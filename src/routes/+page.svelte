<script lang="ts">
	import { onMount } from 'svelte';
	let context: AudioContext;
	const C4 = 264.0;
	const C0 = C4 / 2 ** 4;
	const MAXGAIN = 1;
	const width = 512;
	const height = 256;
	const stringWidth = 1024;
	const stringHeight = 64;
	let analyser: AnalyserNode;
	let canvas: HTMLCanvasElement;
	let tones: Tone[] = [];
	class Tone {
		o: OscillatorNode;
		g: GainNode;
		p: StereoPannerNode;
		playing: boolean;
		key: number;
		name: string;
		octave: number;

		constructor(
			freq: number,
			name: string,
			octave: number,
			context: AudioContext,
			analyser: AnalyserNode
		) {
			this.o = context.createOscillator();
			this.g = context.createGain();
			this.p = new StereoPannerNode(context);
			this.playing = false;
			this.o.frequency.value = freq;
			this.o.connect(this.p);
			this.p.connect(this.g);
			this.o.start();
			this.g.connect(analyser);
			this.g.gain.value = 0;
			tones = [...tones, this];
			this.key = tones.indexOf(this);
			this.name = name;
			this.octave = octave;
		}
	}

	const noteNames = ['C', 'D♭', 'D', 'E♭', 'E', 'F', 'G♭', 'G', 'A♭', 'A', 'B♭', 'B'];

	const makeTonesEqual = (context: AudioContext, analyser: AnalyserNode) => {
		const DIVISIONS = 12;
		return [...Array(DIVISIONS * 8 + 1)].reduce(
			(acc: { [key: string]: number }, _, i) => ({
				...acc,
				[`${noteNames[i % noteNames.length]}${Math.floor(i / noteNames.length)}`]: new Tone(
					C0 * Math.pow(2, i / DIVISIONS),
					noteNames[i % noteNames.length],
					Math.floor(i / noteNames.length),
					context,
					analyser
				).key
			}),
			{}
		);
	};

	const ratiosJust = [
		1 / 1, // C
		16 / 15, // C♯/D♭
		9 / 8, // D
		6 / 5, // D♯/E♭
		5 / 4, // E
		4 / 3, // F
		7 / 5, // F♯/G♭
		3 / 2, // G
		8 / 5, // G♯/A♭
		5 / 3, // A
		16 / 9, // A♯/B♭
		15 / 8 // B
	];

	const ratiosPyth = [
		1 / 1, // C
		2187 / 2048, // C♯/D♭
		9 / 8, // D
		32 / 27, // D♯/E♭
		81 / 64, // E
		4 / 3, // F
		729 / 512, // F♯/G♭
		3 / 2, // G
		6561 / 4096, // G♯/A♭
		27 / 16, // A
		16 / 9, // A♯/B♭
		243 / 128 // B
	];

	const makeTonesJust = (context: AudioContext, analyser: AnalyserNode) => {
		return [...Array(ratiosJust.length * 8 + 1)].reduce(
			(acc: { [key: string]: number }, _, i) => ({
				...acc,
				[`${noteNames[i % noteNames.length]}${Math.floor(i / noteNames.length)}`]: new Tone(
					C0 * ratiosJust[i % ratiosJust.length] * Math.pow(2, Math.floor(i / ratiosJust.length)),
					noteNames[i % noteNames.length],
					Math.floor(i / noteNames.length),
					context,
					analyser
				).key
			}),
			{}
		);
	};

	const noteNamesPersian = [
		'C',
		'D♭',
		'D𝇪',
		'D',
		'E♭',
		'E𝇪',
		'E',
		'F',
		'F𝇩',
		'G♭',
		'G',
		'A♭',
		'A𝇪',
		'A',
		'B♭',
		'B𝇪',
		'B'
	];

	const ratiosPersian = [
		1 / 1, // C
		256 / 243, // C♯/D♭
		2187 / 2048, // D𝇪
		9 / 8, // D
		32 / 27, // D♯/E♭
		19683 / 16384, // E𝇪
		81 / 64, // E
		4 / 3, // F
		1024 / 729, // F𝇩
		729 / 512, // F♯/G♭
		3 / 2, // G
		128 / 81, // G♯/A♭
		6561 / 4096, // A𝇪
		27 / 16, // A
		16 / 9, // A♯/B♭
		59049 / 32768, // B𝇪
		243 / 128 // B
	];

	const makeTonesPersian = (context: AudioContext, analyser: AnalyserNode) => {
		return [...Array(ratiosPersian.length * 8 + 1)].reduce(
			(acc: { [key: string]: number }, _, i) => ({
				...acc,
				[`${noteNamesPersian[i % noteNamesPersian.length]}${Math.floor(i / noteNamesPersian.length)}`]:
					new Tone(
						C0 *
							ratiosPersian[i % ratiosPersian.length] *
							Math.pow(2, Math.floor(i / ratiosPersian.length)),
						noteNamesPersian[i % noteNamesPersian.length],
						Math.floor(i / noteNamesPersian.length),
						context,
						analyser
					).key
			}),
			{}
		);
	};

	const noteNamesChinese = [
		'黃鐘',
		'大呂',
		'太簇',
		'夾鐘',
		'姑洗',
		'仲呂',
		'蕤賓',
		'林鐘',
		'夷則',
		'南呂',
		'無射',
		'應鐘'
	];
	const exponentsChinese = [
		[0, 0],
		[7, 11],
		[2, 3],
		[9, 14],
		[4, 6],
		[11, 17],
		[6, 9],
		[1, 1],
		[8, 12],
		[3, 4],
		[10, 15],
		[5, 7]
	];
	const ratiosChinese = exponentsChinese.map(([r3, r2]) => 3 ** r3 / 2 ** r2);
	const makeTonesChinese = (context: AudioContext, analyser: AnalyserNode) => {
		return [...Array(ratiosChinese.length * 9 + 1)].reduce(
			(acc: { [key: string]: number }, _, i) => ({
				...acc,
				[`${noteNamesChinese[i % noteNamesChinese.length]}${Math.floor(i / noteNamesChinese.length)}`]:
					new Tone(
						C0 *
							ratiosChinese[i % ratiosChinese.length] *
							Math.pow(2, Math.floor(i / ratiosChinese.length)),
						noteNamesChinese[i % noteNamesChinese.length],
						Math.floor(i / noteNamesChinese.length),
						context,
						analyser
					).key
			}),
			{}
		);
	};

	const keyRows = [
		['KeyZ', 'KeyX', 'KeyC', 'KeyV', 'KeyB', 'KeyN', 'KeyM', 'Comma', 'Period', 'Slash'],
		['KeyA', 'KeyS', 'KeyD', 'KeyF', 'KeyG', 'KeyH', 'KeyJ', 'KeyK', 'KeyL', 'Semicolon', 'Quote'],
		[
			'KeyQ',
			'KeyW',
			'KeyE',
			'KeyR',
			'KeyT',
			'KeyY',
			'KeyU',
			'KeyI',
			'KeyO',
			'KeyP',
			'BracketLeft',
			'BracketRight',
			'Backslash'
		],
		[
			'Digit1',
			'Digit2',
			'Digit3',
			'Digit4',
			'Digit5',
			'Digit6',
			'Digit7',
			'Digit8',
			'Digit9',
			'Digit0',
			'Minus',
			'Equal'
		]
	];

	const modes: { [key: string]: string[] } = {
		major: ['C', 'D', 'E', 'F', 'G', 'A', 'B'],
		minor: ['C', 'D', 'E♭', 'F', 'G', 'A♭', 'B♭'],
		بیات_ترک: ['C', 'D', 'E𝇪', 'F', 'G', 'A', 'B♭'],
		نوا: ['C', 'D', 'E𝇪', 'F', 'G', 'A', 'B♭'],
		همایون: ['C', 'D', 'E♭', 'F', 'G', 'A𝇪', 'B'],
		اصفهان: ['C', 'D', 'E𝇪', 'G♭', 'G', 'A', 'B♭'],
		چهارگاه: ['C', 'D𝇪', 'E', 'F', 'G', 'A𝇪', 'B'],
		راست_پنجگاه: ['C', 'D', 'E', 'F', 'G', 'A', 'B♭'],
		羽: ['黃鐘', '夾鐘', '仲呂', '林鐘', '無射'],
		商: ['黃鐘', '太簇', '仲呂', '林鐘', '無射'],
		宫: ['黃鐘', '太簇', '姑洗', '林鐘', '南呂'],
		角: ['黃鐘', '夾鐘', '仲呂', '夷則', '無射'],
		徵: ['黃鐘', '太簇', '仲呂', '林鐘', '南呂']
	};
	const makeKeyMapFromRows = (notes: string[], octave: number) =>
		keyRows.reduce(
			(acc: { [key: string]: string }, row, rowNum) => ({
				...acc,
				...row.reduce(
					(acc: { [key: string]: string }, key, i) => ({
						...acc,
						[key]: `${notes[i % notes.length]}${Math.floor(i / notes.length) + octave + rowNum}`
					}),
					{}
				)
			}),
			{}
		);
	let selectedMode = 'major';
	let keyMap: { [key: string]: string };
	let selectedTemperment = 'persian';
	let temperments: { [key: string]: { [key: string]: number } } = {
		just: {},
		equal: {},
		persian: {},
		chinese: {}
	};
	let tonesTemperment: { [key: string]: number } = {};
	let keyPressed: { [key: string]: boolean } = {};
	let handleKeydown = (e: KeyboardEvent) => {};
	let handleKeyup = (e: KeyboardEvent) => {};

	let playingTones: Tone[] = [];
	$: {
		const gain = MAXGAIN / tones.filter((t) => t.playing).length;
		playingTones = tones
			.map((t) => {
				t.g.gain.setTargetAtTime(t.playing ? gain : 0, context.currentTime, 0.01);
				return t;
			})
			.filter((t) => t.playing);
		context?.resume();
	}
	$: {
		keyMap = makeKeyMapFromRows(modes[selectedMode], 3);
	}
	$: {
		tonesTemperment = temperments[selectedTemperment];
	}

	onMount(() => {
		context = new AudioContext();
		analyser = context.createAnalyser();
		analyser.connect(context.destination);
		temperments = {
			just: makeTonesJust(context, analyser),
			equal: makeTonesEqual(context, analyser),
			persian: makeTonesPersian(context, analyser),
			chinese: makeTonesChinese(context, analyser)
		};
		tonesTemperment = temperments[selectedTemperment];
		handleKeydown = (e: KeyboardEvent) => {
			e.preventDefault();
			if (!keyPressed[e.code]) {
				keyPressed[e.code] = true;
				if (e.code && keyMap[e.code]) {
					tones[tonesTemperment[keyMap[e.code]]].playing = true;
				}
			}
		};
		handleKeyup = (e: KeyboardEvent) => {
			e.preventDefault();
			keyPressed[e.code] = false;
			if (e.code && keyMap[e.code]) {
				tones[tonesTemperment[keyMap[e.code]]].playing = false;
			}
		};
		let bufferLength = analyser.frequencyBinCount;
		let dataArray = new Uint8Array(bufferLength);

		const canvasCtx = canvas.getContext('2d');
		if (canvasCtx) {
			canvasCtx.clearRect(0, 0, width, height);
			const draw = () => {
				const drawVisual = requestAnimationFrame(draw);
				analyser.getByteTimeDomainData(dataArray);
				canvasCtx.fillStyle = 'rgb(200, 200, 200)';
				canvasCtx.fillRect(0, 0, width, height);
				canvasCtx.lineWidth = 2;
				canvasCtx.strokeStyle = 'rgb(0, 0, 0)';
				canvasCtx.beginPath();
				const sliceWidth = (width * 2.0) / bufferLength;
				let x = 0;
				for (var i = 0; i < bufferLength; i++) {
					const v = dataArray[i] / 128.0;
					const y = (v * height) / 2;

					if (i === 0) {
						canvasCtx.moveTo(x, y);
					} else {
						canvasCtx.lineTo(x, y);
					}

					x += sliceWidth;
				}
				if (canvas) {
					canvasCtx.lineTo(canvas.width, canvas.height / 2);
					canvasCtx.stroke();
				}
			};
			draw();
		}
	});
</script>

<svelte:window on:keydown={handleKeydown} on:keyup={handleKeyup} />

<main>
	<canvas bind:this={canvas} {width} {height} />
	<!-- <div
		id="string-container"
		style:inline-size="{stringWidth + stringHeight}px"
		style:block-size="{stringHeight}px"
	>
		<svg
			style:inline-size="{stringWidth}px"
			style:block-size="{stringHeight}px"
			id="string"
			viewBox="0 0 {stringWidth} {stringHeight}"
		>
			<path d="M 0 {stringHeight / 4} v {stringHeight / 2}" />
			<path d="M 0 {stringHeight / 2} h {stringWidth}" />
			<path d="M {stringWidth} {stringHeight / 4} v {stringHeight / 2}" />
			{#each playingTones as tone}
				{@const BASENOTE = tones[tonesTemperment[keyMap[keyRows[0][0]]]].o.frequency.value}
				<path
					d="M {((tone.o.frequency.value - BASENOTE) / (BASENOTE * 4 - BASENOTE)) *
						stringWidth} {stringHeight / 4} v {stringHeight / 2}"
					stroke="red"
				/>
			{/each}
		</svg>
	</div> -->
	<div style="display: flex; flex-direction: row; block-size: 48px;">
		<select bind:value={selectedMode}>
			{#each Object.keys(modes) as mode}<option value={mode}>{mode}</option>{/each}
		</select>
		<select bind:value={selectedTemperment}>
			{#each Object.keys(temperments) as temperment}<option value={temperment}>{temperment}</option
				>{/each}
		</select>

		<div>
			{#each playingTones as t}
				<span>{t.name}{t.octave}: {t.o.frequency.value}</span>
			{/each}
		</div>
	</div>
	<div class="tone-button-container">
		{#each Object.values(tonesTemperment) as t}
			{@const name = tones[t].name + tones[t].octave}
			<input type="checkbox" bind:checked={tones[t].playing} />
			<div class="tone-button">{name}</div>
			<!-- <button
				on:touchstart={() => (tones[t].playing = true)}
				on:touchend={() => (tones[t].playing = false)}
			>
				<span style="user-select: none;">{tones[t].name}{tones[t].octave}</span>
			</button> -->
		{/each}
	</div>
	<!-- <div class="tone-grid">
		{#each Object.values(keyMap) as noteName}
			{#if tonesTemperment[noteName]}
				<div>
					{tones[tonesTemperment[noteName]].name}
					<input type="checkbox" bind:checked={tones[tonesTemperment[noteName]].playing} />
					<input
						type="number"
						bind:value={tones[tonesTemperment[noteName]].o.frequency.value}
						min="0"
						max="7040"
						step="any"
					/>
				</div>
			{/if}
		{/each}
	</div> -->
</main>

<style>
	main {
		font-family: 'Noto Sans', 'Noto Music';
	}
	#string-container {
		border: 1px black solid;
		display: flex;
		justify-content: center;
	}
	#string {
		stroke: black;
	}
	.tone-grid {
		display: grid;
		grid-template-rows: repeat(12, 1fr);
	}

	.tone-button-container {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
	}
	.tone-button {
		font-family: 'Noto Sans', 'Noto Music';
		inline-size: 36px;
		block-size: 36px;
		background-color: lightblue;
		border-radius: 8px;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	input[type='checkbox'] {
		display: none;
	}

	input[type='checkbox']:checked + .tone-button {
		background-color: pink;
	}
</style>
