<script>
	import { onMount } from "svelte";
	let Islands = [];
	let dailyNumber = [
		14, 30, 38, 27, 45, 43, 59, 9, 20, 34, 46, 69, 32, 40, 52, 42, 26, 57,
		18, 56, 74, 78, 48, 41, 62, 55, 4, 25, 12, 65, 10, 66, 21, 68, 28, 73,
		51, 71, 47, 77, 7, 22, 31, 5, 54, 50, 23, 3, 0, 1, 75, 64, 79, 33, 58,
		29, 63, 49, 8, 67, 13, 44, 37, 60, 17, 6, 72, 16, 24, 2, 35, 11, 70, 19,
		53, 61, 36, 15, 39, 76,
	];

	let answer;
	let imgsrc;
	let attempts;
	let day;

	let clicked;
	let focused;
	let blurred = false;

	let guessno = 1;
	let guess;

	let buttonText = "Guess";

	let boxes = ["", "", "", "", "", ""];
	let guessParagraph = ["", "", "", "", "", ""];
	let gpClass = ["", "", "", "", "", ""];
	let guessDirection = ["", "", "", "", "", ""];

	function setup() {
		var now = new Date();
		var start = new Date(now.getFullYear(), 0, 0);
		var diff = now - start;
		var oneDay = 1000 * 60 * 60 * 24;
		day = Math.floor(diff / oneDay);
		while (day > 79) {
			day -= 79;
		}
		answer = Islands[dailyNumber[day]];
		imgsrc = answer.Image;
		guessno = 1;
	}

	function loadIslands() {
		fetch("islands.json").then((res) => {
			res.json().then((data) => {
				Islands = data;

				var list = document.getElementById("options");
				Islands.forEach(function (item) {
					var option = document.createElement("option");
					option.value = item["Name"];
					list.appendChild(option);
				});

				fill();

				setup();
			});
		});
	}

	onMount(async () => {
		loadIslands();
	});

	function isIsland(x) {
		if (Islands.find((i) => i.Name === x)) {
			return true;
		}
		return false;
	}

	function getIsland(x) {
		return Islands.find((i) => i.Name === x);
	}

	function isWin(x) {
		return answer["Name"] === x;
	}

	function is_same_region(x) {
		return answer["Region"] === x["Region"];
	}

	function direction(x) {
		var direction;

		if (answer["LocationY"] - x["LocationY"] < 0) {
			direction = "N";

			if (
				answer["LocationX"].charCodeAt(0) -
					x["LocationX"].charCodeAt(0) >
				0
			) {
				direction += "E";
			} else if (
				answer["LocationX"].charCodeAt(0) -
					x["LocationX"].charCodeAt(0) <
				0
			) {
				direction += "W";
			}
		} else if (answer["LocationY"] - x["LocationY"] > 0) {
			direction = "S";

			if (
				answer["LocationX"].charCodeAt(0) -
					x["LocationX"].charCodeAt(0) >
				0
			) {
				direction += "E";
			} else if (
				answer["LocationX"].charCodeAt(0) -
					x["LocationX"].charCodeAt(0) <
				0
			) {
				direction += "W";
			}
		}
		if (answer["LocationY"] === x["LocationY"]) {
			if (
				answer["LocationX"].charCodeAt(0) -
					x["LocationX"].charCodeAt(0) >
				0
			) {
				direction = "E";
			} else if (
				answer["LocationX"].charCodeAt(0) -
					x["LocationX"].charCodeAt(0) <
				0
			) {
				direction = "W";
			}
		}

		return direction;
	}

	function clear() {
		guess = "";
	}

	function blur() {
		blurred = true;
	}

	function click() {
		clicked = true;
		if (focused) {
			if (!blurred) {
				clear();
				focused = false;
				return;
			}
		}
	}
	function focus() {
		focused = true;
		if (clicked) {
			if (!blurred) {
				clear();
				clicked = false;
				return;
			}
		}
		blurred = false;
	}

	function fill() {
		guess = Islands[0]["Name"];
	}
	function showAnswer() {
		alert("The answer was " + answer.Name + ".");
	}
	function shareResults(x) {
		navigator.clipboard.writeText(x);
	}

	function guessButton() {
		if (guessno <= 6) {
			if (isIsland(guess)) {
				if (isWin(guess)) {
					gpClass[guessno - 1] = "win";
					boxes[guessno - 1] = "🟩";
					guessParagraph[guessno - 1] = guess;
					attempts = guessno;
					guessno = 9;
					buttonText = "Share";
					blurred = false;
					return;
				}

				let guessIsland = getIsland(guess);

				if (is_same_region(guessIsland)) {
					gpClass[guessno - 1] = "close";
					boxes[guessno - 1] = "🟨";
				} else {
					boxes[guessno - 1] = "⬜";
				}

				const index = Islands.indexOf(guessIsland);
				Islands.splice(index, 1);

				var list = document.getElementById("options");
				console.log(list.children[index]);
				list.children[index].remove();

				guessParagraph[guessno - 1] = guess + " ";
				guessDirection[guessno - 1] = direction(guessIsland);
				guessno += 1;
				attempts = guessno;
				if (guessno === 7) {
					buttonText = "Answer";
					blurred = false;
					return;
				}
				fill();
			}
		} else if (guessno === 7) {
			showAnswer();
			buttonText = "Share";
			guessno = 9;
			blurred = false;
			return;
		} else if (guessno === 8) {
			document.location.reload();
		} else if (guessno === 9) {
			if (attempts === 7) {
				attempts = "x";
			}
			alert("Copied to clipboard.");
			shareResults(
				`Sotdle ${day} ${attempts}/6 \n${boxes[0]}${boxes[1]}${boxes[2]}${boxes[3]}${boxes[4]}${boxes[5]}\nhttps://daily.sotdle.xyz`
			);
		}
	}
</script>

<datalist id="options" />

<main>
	<header>
		<img class="logo" src="Sotdle Daily.png" alt="Sotdle" />
		<h6>
			Alex Burneikis <a
				href="https://github.com/alexburneikis/sotdle-daily">Github</a
			>,
			<a
				href="https://github.com/AlexBurneikis/Sotdle-daily/blob/main/README.md"
				>Help</a
			>,
			<a href="https://sotdle.xyz">Infinite</a>
		</h6>
	</header>
	<img src={imgsrc} alt="island" />
	<body>
		<input
			bind:value={guess}
			list="options"
			on:focus={focus}
			on:blur={blur}
			on:click={click}
		/>
		<button class="disable-dbl-tap-zoom" on:click={guessButton}
			>{buttonText}</button
		>
		<p class={gpClass[0]}>
			{guessParagraph[0]}<span>{guessDirection[0]}</span>
		</p>
		<p class={gpClass[1]}>
			{guessParagraph[1]}<span>{guessDirection[1]}</span>
		</p>
		<p class={gpClass[2]}>
			{guessParagraph[2]}<span>{guessDirection[2]}</span>
		</p>
		<p class={gpClass[3]}>
			{guessParagraph[3]}<span>{guessDirection[3]}</span>
		</p>
		<p class={gpClass[4]}>
			{guessParagraph[4]}<span>{guessDirection[4]}</span>
		</p>
		<p class={gpClass[5]}>
			{guessParagraph[5]}<span>{guessDirection[5]}</span>
		</p>
	</body>
</main>

<style>
	@font-face {
		font-family: "Sotfont2";
		src: url(/sot_fonts/font2.ttf);
	}
	@font-face {
		font-family: "Sotfont3";
		src: url(/sot_fonts/font3.ttf);
	}
	@font-face {
		font-family: "Sotfont4";
		src: url(/sot_fonts/font4.ttf);
	}
	span {
		font-family: Sotfont3;
	}

	main {
		margin: 0 auto;

		padding: 0;
		padding-top: 0;

		text-align: center;
	}
	.logo {
		height: calc(6vh - env(safe-area-inset-bottom));
		min-height: 30px;
		width: auto;
		margin: auto;
	}
	h6 {
		margin: 0;
		font-family: Sotfont4;
	}
	body {
		margin-top: 2px;
		padding: 0;
		text-align: center;
		height: calc(20vh - env(safe-area-inset-bottom));
	}
	p {
		font-family: Sotfont2;
		font-size: calc(1.5vh - env(safe-area-inset-bottom));

		margin: auto;
		margin-bottom: 2px;

		width: calc(60vmin - env(safe-area-inset-bottom));
		height: 12.5%;

		border-radius: 5px;

		background-color: rgb(181, 165, 153);
	}

	img {
		height: calc(70vmin - env(safe-area-inset-bottom));
		display: block;
		margin: auto;
		border-radius: 8px;
	}

	input {
		vertical-align: top;
		padding: 0;
		padding-left: 5px;
		border-radius: 5px;
		height: 16%;
		margin: 0;
		width: calc(45vmin - env(safe-area-inset-bottom));
		font-size: calc(2.4vmin - env(safe-area-inset-bottom));
		border-radius: 5px;
		border: 0;
		font-family: Sotfont3;
		background-color: rgb(181, 165, 153);
	}
	button {
		margin-bottom: 2px;
		display: inline;
		vertical-align: top;
		border-radius: 5px;
		height: 16%;
		width: calc(14vmin - env(safe-area-inset-bottom));
		font-size: calc(2.5vmin - env(safe-area-inset-bottom));
		padding: 0;
		border: 0;
		background-color: rgb(181, 165, 153);
		font-family: Sotfont3;
	}

	.win {
		color: black;
		background-color: rgb(0, 255, 0);
	}
	.close {
		color: black;
		background-color: yellow;
	}

	.disable-dbl-tap-zoom {
		touch-action: manipulation;
	}
</style>
