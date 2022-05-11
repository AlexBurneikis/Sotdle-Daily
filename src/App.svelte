<script>
	import { onMount } from "svelte";
	let Islands = [];

	let answer;
	let imgsrc;

	let clicked;
	let focused;
	let blurred = false;

	let guessno = 1;
	let guess;

	let buttonText = "Guess";

	let guessParagraph = ["", "", "", "", "", ""];
	let gpClass = ["", "", "", "", "", ""];
	let guessDirection = ["", "", "", "", "", ""];

	function setup() {
		answer = Islands[Math.floor(Math.random() * Islands.length)];
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

	function guessButton() {
		if (guessno <= 6) {
			if (isIsland(guess)) {
				if (isWin(guess)) {
					gpClass[guessno - 1] = "win";
					guessParagraph[guessno - 1] = guess;
					guessno = 8;
					buttonText = "Again";
					blurred = false;
					return;
				}

				let guessIsland = getIsland(guess);

				if (is_same_region(guessIsland)) {
					gpClass[guessno - 1] = "close";
				}

				const index = Islands.indexOf(guessIsland);
				Islands.splice(index, 1);

				var list = document.getElementById("options");
				console.log(list.children[index]);
				list.children[index].remove();

				guessParagraph[guessno - 1] = guess + " ";
				guessDirection[guessno - 1] = direction(guessIsland);

				guessno += 1;
				if (guessno === 7) {
					buttonText = "Answer";
					blurred = false;
					return;
				}
				fill();
			}
		} else if (guessno === 7) {
			showAnswer();
			buttonText = "Again";
			guessno = 8;
			blurred = false;
			return;
		} else if (guessno === 8) {
			document.location.reload();
		}
	}
</script>

<datalist id="options" />

<main>
	<header>
		<img class="logo" src="Sotdle.png" alt="Sotdle" />
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
			Alex Burneikis 2022 <a
				href="https://github.com/alexburneikis/sotdle">Github</a
			>,
			<a
				href="https://github.com/AlexBurneikis/Sotdle/blob/main/README.md"
				>Help</a
			>
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
