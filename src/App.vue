<template>
	<div class="wrapper">
		<div class="figures">

		</div>
		<div class="letters">
			<ul class="letters__container">
				<li class="letters__letter" v-for="(letter) in lettersDOM" :key="letter['index']" 
					:style="{
						transform: `translate(${letter['x']}px, ${letter['y']}px)`
					}"
				>
					{{ letter['letter'] }} 
				</li>
			</ul>
		</div>
	</div>
</template>

<script setup>
	import { ref, onMounted } from 'vue';

	const lettersDOM = ref([])
	const rotateAngle = ref(0)
	const radius = ref(250)
	const xStep = ref(30)
	const yStep = ref(50)

	let timeSet = generateAllTimeWords()
	let wordsSet = Object.values(timeSet)
	let lettersSet = {}

	onMounted(() => {
		if (window.innerWidth < 650) {
			radius.value = 125
			xStep.value = 15
			yStep.value = 25
		} else {
			radius.value = 250
			xStep.value = 30
			yStep.value = 50
		}
		for (let i = 0; i < wordsSet.length; i++) {
			wordsSet = wordsSet.map(elem => elem.toUpperCase())
			let word = wordsSet[i].split('')
			let wordLetters = {}
			for (let j = 0; j < word.length; j++) {
				if (wordLetters[word[j]]) {
					wordLetters[word[j]] += 1
				} else {
					if (word[j] != ' ')
					wordLetters[word[j]] = 1
				}
			}
			let keys = Object.keys(wordLetters)
			for (let j = 0; j < keys.length; j++) {
				if (lettersSet[keys[j]]) {
					if (lettersSet[keys[j]] < wordLetters[keys[j]]) {
						lettersSet[keys[j]] = wordLetters[keys[j]]
					}
				} else {
					lettersSet[keys[j]] = wordLetters[keys[j]]
				}
			}
		}

		let keys = Object.keys(lettersSet)
		let itterator = 0
		for (let i = 0; i < keys.length; i++) {
			for (let j = 0; j < lettersSet[keys[i]]; j++) {
				lettersDOM.value.push({index: itterator, letter: keys[i], x: 0, y: 0, isPartOfWord: false})
				itterator++
			}
		}
		lettersDOM.value.sort(() => Math.random() - 0.5)

		calculateCoordinates()
		setInterval(() => {
			rotateAngle.value += 4
			let date = new Date()
			let currentTime = timeSet[`${date.getHours()}:${date.getMinutes()}`]
			getLettersInWord(currentTime)
			calculateCoordinates()
		}, 1000)

		window.addEventListener('resize', () => {
			if (window.innerWidth < 650) {
				radius.value = 125
				xStep.value = 15
				yStep.value = 25
			} else {
				radius.value = 250
				xStep.value = 30
				yStep.value = 50
			}
		})

	})

	const calculateCoordinates = () => {
		const lettersOnCircle = lettersDOM.value.filter(el => !el.isPartOfWord).length
		let angle = 360 / lettersOnCircle
		let angleHash = 0

		let date = new Date()
		let currentTime = timeSet[`${date.getHours()}:${date.getMinutes()}`]
		let lettersPositions = getLettersPositions(currentTime)

		for (let i = 0; i < lettersDOM.value.length; i++) {
			if (!lettersDOM.value[i].isPartOfWord) {
				lettersDOM.value[i].x = radius.value * Math.cos(((angleHash) + rotateAngle.value) * Math.PI / 180) - 30
				lettersDOM.value[i].y = radius.value * Math.sin(((angleHash) + rotateAngle.value) * Math.PI / 180) - 15
				angleHash += angle
			} else {
				for (let j = 0; j < lettersPositions.length; j++) {
					if (lettersDOM.value[i].letter == lettersPositions[j].letter.toUpperCase() && !lettersPositions[j].isUsed) {
						lettersDOM.value[i].x = lettersPositions[j].x
						lettersDOM.value[i].y = lettersPositions[j].y
						lettersPositions[j].isUsed = true
						break
					}
				}
			}
		}
	}

	const getLettersInWord = (string) => {
		string = string.split('')
		for (let i = 0; i < lettersDOM.value.length; i++) {
			lettersDOM.value[i].isPartOfWord = false
		}
		for (let i = 0; i < string.length; i++) {
			for (let j = 0; j < lettersDOM.value.length; j++) {
				if (string[i].toUpperCase() == lettersDOM.value[j].letter && lettersDOM.value[j].isPartOfWord == false) {
					lettersDOM.value[j].isPartOfWord = true
					break
				}
			}
		}
	}

	const getLettersPositions = (string) => {
		string = string.split(' ')

		let positions = []
		let xShift = 0
		let yShift = 0

		if (string.length == 3 ) {
			yShift = -yStep.value - 15
		} else {
			yShift = (-yStep.value / 2) - 15
		}

		for (let i = 0; i < string.length; i++) {
			xShift = ((-string[i].length / 2) + 0.5) * xStep.value - 30
			for (let j = 0; j < string[i].length; j++) {
				positions.push({letter: string[i][j], x: xShift, y: yShift, isUsed: false})
				xShift += xStep.value
			}
			yShift += yStep.value
		}
		return positions
	}

	function numberToWords(num) {
		const ones = ['zero','one','two','three','four','five','six','seven','eight','nine','ten', 'eleven','twelve','thirteen','fourteen','fifteen','sixteen','seventeen','eighteen','nineteen'];
		const tens = ['','','twenty','thirty','forty','fifty'];

		if (num < 20) return ones[num];
		const ten = Math.floor(num / 10);
		const one = num % 10;
		return tens[ten] + (one ? '-' + ones[one] : '');
	}

	function timeToWords(hour, minute) {
		if (minute === 0) {
			return `${numberToWords((hour % 12) || 12)} o’clock`;
		} else if (minute === 15) {
			return `quarter past ${numberToWords((hour % 12) || 12)}`;
		} else if (minute === 30) {
			return `half past ${numberToWords((hour % 12) || 12)}`;
		} else if (minute === 45) {
			return `quarter to ${numberToWords(((hour + 1) % 12) || 12)}`;
		} else if (minute < 30) {
			return `${numberToWords(minute)} past ${numberToWords((hour % 12) || 12)}`;
		} else {
			return `${numberToWords(60 - minute)} to ${numberToWords(((hour + 1) % 12) || 12)}`;
		}
	}

	function generateAllTimeWords() {
		const times = {};
		for (let hour = 0; hour < 24; hour++) {
			for (let minute = 0; minute < 60; minute++) {
				times[`${hour}:${minute}`] = timeToWords(hour, minute)
			}
		}
		return times;
	}
</script>

<style scoped>
	.wrapper {
		width: 100%;
		height: 100vh;
		position: relative;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.letters {
		width: 500px;
		height: 500px;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.letters__container {
		position: relative;
	}

	.letters__letter {
		list-style: none;
		position: absolute;
		color: white;
		font-size: 2rem;
		font-family: 'Courier New', Courier, monospace;
		transition: 1s ease-in-out;
	}

	@media (width < 650px) {
		.letters__letter {
			font-size: 1rem;
		}
	}
</style>