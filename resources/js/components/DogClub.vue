<template>
<div class="game-grid">
	<castleBoard :player="player" :boardData="boardData" :board="board" :deck="decks[player.color]" :helpers="helpers"></castleBoard>
	<deckBox :player="player" :board="board" :deck="decks[player.color]" :helpers="helpers"></deckBox>
</div>
</template>

<script>
import castleBoard from './modules/Board.vue'
import deckBox from './modules/Deck.vue'

let BoardData = {
	coords: null,
	deck: null
};
BoardData.coords = require('./data/boarddata.js').getBoardDataCoords();
BoardData.deck = require('./data/deck.js').default;

console.log(BoardData);

export default {
	name: 'DogClub',
	components: {
		castleBoard,
		deckBox
	},
	data() {
		let self = this;
		return {
			boardData: BoardData,
			board: {
				history: [],
				currentStepInfo: 'Pick Card from Hand!',
				pins: {},
				moveOptions: {},
				holdingWalls: {}
			},
			cards: null,
			decks: {
				red: {
					cards: [],
					picked: null
				},
				green: {
					cards: [],
					picked: null
				},
				orange: {
					cards: [],
					picked: null
				},
				blue: {
					cards: [],
					picked: null
				}
			},
			round: 0,
			cardsToGive: 6,
			players: ['red', 'green', 'orange', 'blue'],
			startingPlayer: 1,
			turnPlayer: 1,
			player: {
				color: 'red'
			},
			helpers: {
				playerTurnFinished: function(player) {
					console.log('playerTurnFinished', player.color);
					if(self.isRoundOver()) {
						console.log('Round IS Over');
						self.giveCards(self.cardsToGive, self.startingPlayer);
					}else {
						console.log('Round NOT Over');
						self.nextPlayersTurn();
					}

					// HACK: remove this that is a SOLO mode hack
					self.player.color = self.players[self.turnPlayer];
					return;
				}
			}
		}
	},
	mounted() {
		let self = this;
		console.log('#Root Mounted');
		self.initNewGame();
	},
	methods: {
		initNewGame: function() {
			let self = this;
			console.log('initNewGame');

			self.startingPlayer = Math.floor(Math.random() * 4);
			self.turnPlayer = self.startingPlayer;
			// HACK: remove()
			self.player.color = self.players[self.turnPlayer];

			self.cards = self.shuffle(self.boardData.deck);

			self.giveCards(self.cardsToGive, self.startingPlayer);
		},
		shuffle: function(array) {
			var currentIndex = array.length,
				temporaryValue, randomIndex;

			// While there remain elements to shuffle...
			while (0 !== currentIndex) {

				// Pick a remaining element...
				randomIndex = Math.floor(Math.random() * currentIndex);
				currentIndex -= 1;

				// And swap it with the current element.
				temporaryValue = array[currentIndex];
				array[currentIndex] = array[randomIndex];
				array[randomIndex] = temporaryValue;
			}

			return [...array];
		},
		giveCards: function(count, startWith) {
			let self = this;
			console.log('giving Cards', count, startWith);
			let currentPlayer = startWith;
			self.turnPlayer = currentPlayer;

			for (var runIndex = 0; runIndex < count; runIndex++) {
				console.log('run', runIndex);
				for (let player of self.players) {
					self.giveCardToPlayer(currentPlayer);
					currentPlayer = (currentPlayer+1) < 4 ? currentPlayer+1 : 0;
				}
			}

			self.startingPlayer = (self.startingPlayer+1) < 4 ? self.startingPlayer+1 : 0;
			self.cardsToGive = (self.cardsToGive-1) > 1 ? self.cardsToGive-1 : 5;
			self.round++;
		},
		giveCardToPlayer: function(player) {
			let self = this;
			console.log('giveCardToPlayer', self.players[player]);
			let color = self.players[player];
			self.decks[color].cards.push(self.cards[0]);
			self.cards.shift();

			if (self.cards.length === 0) {
				self.cards = self.shuffle(self.boardData.deck);
			}
		},
		isRoundOver: function() {
			let self = this;
			console.log('checkIfRoundIsOver');

			let isOver = true;
			if(self.decks.red.cards.length > 0) {
				isOver = false;
			} else if(self.decks.green.cards.length > 0) {
				isOver = false;
			} else if(self.decks.orange.cards.length > 0) {
				isOver = false;
			} else if(self.decks.blue.cards.length > 0) {
				isOver = false;
			}

			return isOver;
		},
		nextPlayersTurn: function() {
			let self = this;
			console.log('nextPlayersTurn');

			self.turnPlayer = (self.turnPlayer+1) < 4 ? self.turnPlayer+1 : 0;
			if(self.decks[self.players[self.turnPlayer]].cards.length == 0) {
				self.nextPlayersTurn();
				return;
			};
		}
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
		grid-template-rows: 80% 20%;
		grid-column-gap: 0;
		grid-row-gap: 0;
	}
</style>
