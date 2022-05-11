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

	let boxes = ["", "", "", "", "", ""]
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
				}
				else {
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
			attempts = 'x'
			}
			alert('Copied to clipboard.')
			shareResults(`Sotdle ${day} ${attempts}/6 \n${boxes[0]}${boxes[1]}${boxes[2]}${boxes[3]}${boxes[4]}${boxes[5]}\nhttps://daily.sotdle.xyz`);
		}
	}
</script>

<datalist id="options" />

<main>
	<header>
		<img class="logo" src="Sotdle Daily.png" alt="Sotdle" />
	</header>
	<body>
		<img src={imgsrc} alt="island" />
		<br />
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
		<div class="input">
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
		</div>
	</body>
	<footer>
		<h6>
			Alex Burneikis <a
				href="https://github.com/alexburneikis/sotdle-daily">Github</a
			>,
			<a
				href="https://github.com/AlexBurneikis/Sotdle-daily/blob/main/README.md"
				>Help</a
			>,
			<a
				href="https://sotdle.xyz"
				>Infinite</a>
		</h6>
	</footer>
</main>

<style>
	.disable-dbl-tap-zoom {
		touch-action: manipulation;
	}

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
	footer {
		height: 40px;
		bottom: 0;
		position: fixed;
		width: 240px;
		font-family: Sotfont4;
	}

	main {
		text-align: center;
		padding: 1em;
		width: 240px;
		margin: 0 auto;
	}
	header {
		height: 50px;
	}
	.logo {
		height: 50px;
		width: auto;
	}
	.input {
		width: 240px;
		font-family: Sotfont3;
	}
	p {
		font-family: Sotfont2;
		font-size: 110%;
		margin-top: 0px;
		margin-bottom: 5px;
		background-color: rgb(181, 165, 153);
		width: 240px;
		height: 25px;
		border-radius: 5px;
	}
	img {
		width: 240px;
		border-radius: 8px;
	}
	input {
		margin: 0px;
		margin-top: 5px;
		width: 175px;
		height: 30px;
		border-radius: 5px;
		border: 0;
		background-color: rgb(181, 165, 153);
	}

	button {
		padding: 0;
		margin: 0px;
		margin-top: 5px;
		width: 60px;
		height: 30px;
		border: 0;
		border-radius: 5px;
		background-color: rgb(181, 165, 153);
	}

	.win {
		color: black;
		background-color: rgb(0, 255, 0);
	}

	.close {
		color: black;
		background-color: yellow;
	}
</style>
