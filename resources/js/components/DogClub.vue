<template>
	<div class="game-grid">
		<castleBoard :player="player" :boardData="boardData" :board="board" :deck="deck"></castleBoard>
		<deckBox :player="player" :board="board" :deck="deck"></deckBox>
	</div>
</template>

<script>
import castleBoard from './modules/Board.vue'
import deckBox from './modules/Deck.vue'

let BoardData = { coords: null };
BoardData.coords = require('./data/boarddata.js').getBoardDataCoords();
console.log(BoardData);

export default {
	name: 'DogClub',
	components: {
		castleBoard,
		deckBox
	},
	data () {
		return {
			boardData: BoardData,
			board: {
				history: [],
				currentStepInfo: 'Pick Card from Hand!',
				pins: {},
				moveOptions: {},
				holdingWalls: {}
			},
			deck: {
				cards: [
					{
						id: "spades_ace",
						value: 'A',
						moves: ['out', 1, 11]
					},
					{
						id: "hearts_king",
						value: 'K',
						moves: ['out', 13]
					},
					{
						id: "spades_jack",
						value: 'B',
						moves: ['swap']
					},
					{
						id: "diamonds_six",
						value: '6',
						moves: [6]
					},
					{
						id: "hearts_seven",
						value: '7',
						moves: [1, 2, 3, 4, 5, 6, 7]
					},
					{
						id: "clubs_four",
						value: '4',
						moves: [-4, 4]
					}
				],
				picked: null
			},
			player: {
				color: [
					'red',
					'orange'
				]
			}
		}
	},
	mounted() {
		console.log('#Root Mounted');
	},
	methods: {
		//
	}
}
</script>

<style lang="scss">
	html, body {
		width: 100vw;
		height: 100vh;
		max-width:100vw;
		max-height: 100vh;
		margin: 0;
		overflow: hidden;
	}
	body {
		background:
		radial-gradient(black 15%, transparent 16%) 0 0,
		radial-gradient(black 15%, transparent 16%) 8px 8px,
		radial-gradient(rgba(255,255,255,.1) 15%, transparent 20%) 0 1px,
		radial-gradient(rgba(255,255,255,.1) 15%, transparent 20%) 8px 9px;
		background-color:#282828;
		background-size:16px 16px;
	}


	.game-grid {
		height: calc(100vh - 55px);
		display: grid;
		grid-template-columns: 1fr;
		grid-template-rows: 8fr 2fr;
		grid-column-gap: 0px;
		grid-row-gap: 0px;
	}
</style>
