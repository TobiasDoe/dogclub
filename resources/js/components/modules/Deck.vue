<template>
	<div class="d-flex justify-content-center">
		<div class="deck-box"
			v-if="deck.cards.length > 0"
			v-bind:class="{
				'six-cards': deck.cards.length >= 6,
				'five-cards': deck.cards.length === 5,
				'four-cards': deck.cards.length === 4,
				'three-cards': deck.cards.length === 3,
				'two-cards': deck.cards.length === 2,
				'one-cards': deck.cards.length === 1
			}">

			<img v-for="(card, index) in deck.cards" class="deck-card"
				v-bind:id="'card-' + card.id"
				v-bind:src="'images/cards/'+ card.id +'.png'"
				v-bind:alt="'cardMissing: ' + card.id"
					v-bind:class="{ active: deck.picked != null && deck.picked.id === card.id }"
				v-on:click="pickCard(card)">
		</div>
		<div class="actions">
			<button type="button" name="button" @click="cantMove();">Can't Move!</button>
		</div>
	</div>
</template>

<script>
export default {
	props: ['player', 'board', 'deck', 'helpers'],
	data () {
		return {
			//
		}
	},
	mounted() {
		console.log('#DeckBox Mounted');
	},
	methods: {
		pickCard: function(card) {
			var self = this;
			self.deck.picked = self.deck.picked != null && self.deck.picked.id === card.id ? null : card;
			self.board.moveOptions = {};
			self.board.focusedPin = null;
			if (self.deck.picked) {
				if (card.value == "K" || card.value == "A" || card.value == "K" || card.value == "J") {
					self.showStartMove();
				}
				self.board.currentStepInfo = "Pick a Pin!";
				self.preCheckOptions();
			} else {
				self.board.currentStepInfo = "Pick Card from Hand!";
			}
		},
		showStartMove: function() {
			console.log('showStartMove');
			let self = this;
			let defaultHomePinHoles = {
				red: 1,
				green: 16,
				orange: 31,
				blue: 46
			}
			let moves = {};
			moves[defaultHomePinHoles[self.player.color]] = {
				from: 0,
				move: true
			};

			self.board.moveOptions = moves;
		},
		preCheckOptions: function() {
			let self = this;
			console.log("Precheck");
			let check = '.active.' + self.player.color +', .active.' + self.player.color[1];
			let elements = $(check);
			if (elements.length === 1) {
				elements.trigger('click');
			}
		},
		cantMove: function() {
			let self = this;
			console.log('cantMove');

			self.deck.cards = [];
			self.helpers.playerTurnFinished(self.player);
		}
	}
}
</script>

<style lang="scss">
	.actions {
		position: absolute;
		left: 0;
		bottom: 0;
		padding: .66rem;
	}

	.deck-box {
		display: inline-flex;
		margin-top: 2.33rem;
		position: relative;
		margin-left: 55%;

		.deck-card {
			cursor: pointer;
			position: relative;
			border-radius: 8px;
			box-shadow: 0 0 15px rgba(0, 0, 0, 0.6);
			transition: all .25s;
			height: 312px;
			width: 209px;

			user-drag: none;
			user-select: none;
			-moz-user-select: none;
			-webkit-user-drag: none;
			-webkit-user-select: none;
			-ms-user-select: none;

			&:hover {
				top: 20px !important;
				box-shadow: 0 0 25px rgba(0, 0, 0, 0.6);
			}
			&.active, &:active {
				top: -20px !important;
				box-shadow: 0 0 25px rgba(0, 0, 0, 0.6);
				// z-index: 10;
			}
		}

		&.six-cards {

			.deck-card {

				&:nth-child(1) {
					top: 60px;
					transform: rotate(-16deg);
					&:hover, &.active {
						transform: rotate(-16deg) scale(1.05);
					}
				}
				&:nth-child(2) {
					top: 53px;
					left: -150px;
					transform: rotate(-8deg);
					&:hover, &.active {
						transform: rotate(-8deg) scale(1.05);
					}
				}
				&:nth-child(3) {
					top: 45px;
					left: -300px;
					transform: rotate(-4deg);
					&:hover, &.active {
						transform: rotate(-4deg) scale(1.05);
					}
				}
				&:nth-child(4) {
					top: 45px;
					left: -450px;
					transform: rotate(4deg);
					&:hover, &.active {
						transform: rotate(4deg) scale(1.05);
					}
				}
				&:nth-child(5) {
					top: 50px;
					left: -600px;
					transform: rotate(10deg);
					&:hover, &.active {
						transform: rotate(10deg) scale(1.05);
					}
				}
				&:nth-child(6) {
					top: 65px;
					left: -750px;
					transform: rotate(16deg);
					&:hover, &.active {
						transform: rotate(16deg) scale(1.05);
					}
				}

			}

		}

		&.five-cards {
			margin-left: 47%;

			.deck-card {

				&:nth-child(1) {
					top: 50px;
					transform: rotate(-12deg);
					&:hover, &.active {
						transform: rotate(-16deg) scale(1.05);
					}
				}
				&:nth-child(2) {
					top: 45px;
					left: -150px;
					transform: rotate(-4deg);
					&:hover, &.active {
						transform: rotate(-8deg) scale(1.05);
					}
				}
				&:nth-child(3) {
					top: 40px;
					left: -300px;
					transform: rotate(0deg);
					&:hover, &.active {
						transform: rotate(0deg) scale(1.05);
					}
				}
				&:nth-child(4) {
					top: 45px;
					left: -450px;
					transform: rotate(8deg);
					&:hover, &.active {
						transform: rotate(8deg) scale(1.05);
					}
				}
				&:nth-child(5) {
					top: 55px;
					left: -600px;
					transform: rotate(16deg);
					&:hover, &.active {
						transform: rotate(16deg) scale(1.05);
					}
				}

			}

		}

		&.four-cards {
			margin-left: 56%;

			.deck-card {

				&:nth-child(1) {
					top: 53px;
					left: -150px;
					transform: rotate(-8deg);
					&:hover, &.active {
						transform: rotate(-8deg) scale(1.05);
					}
				}
				&:nth-child(2) {
					top: 45px;
					left: -300px;
					transform: rotate(-4deg);
					&:hover, &.active {
						transform: rotate(-4deg) scale(1.05);
					}
				}
				&:nth-child(3) {
					top: 45px;
					left: -450px;
					transform: rotate(4deg);
					&:hover, &.active {
						transform: rotate(4deg) scale(1.05);
					}
				}
				&:nth-child(4) {
					top: 50px;
					left: -600px;
					transform: rotate(10deg);
					&:hover, &.active {
						transform: rotate(10deg) scale(1.05);
					}
				}

			}

		}

		&.three-cards {
			margin-left: 47%;

			.deck-card {

				&:nth-child(1) {
					top: 45px;
					left: -150px;
					transform: rotate(-4deg);
					&:hover, &.active {
						transform: rotate(-8deg) scale(1.05);
					}
				}
				&:nth-child(2) {
					top: 40px;
					left: -300px;
					transform: rotate(0deg);
					&:hover, &.active {
						transform: rotate(0deg) scale(1.05);
					}
				}
				&:nth-child(3) {
					top: 45px;
					left: -450px;
					transform: rotate(8deg);
					&:hover, &.active {
						transform: rotate(8deg) scale(1.05);
					}
				}

			}

		}

		&.two-cards {
			margin-left: 57%;

			.deck-card {

				&:nth-child(1) {
					top: 45px;
					left: -300px;
					transform: rotate(-4deg);
					&:hover, &.active {
						transform: rotate(-4deg) scale(1.05);
					}
				}
				&:nth-child(2) {
					top: 45px;
					left: -450px;
					transform: rotate(4deg);
					&:hover, &.active {
						transform: rotate(4deg) scale(1.05);
					}
				}

			}

		}

		&.one-cards {
			margin-left: 45%;

			.deck-card {

				&:nth-child(1) {
					top: 45px;
					left: -300px;
					transform: rotate(0deg);
					&:hover, &.active {
						transform: rotate(0deg) scale(1.05);
					}
				}

			}

		}

	}
</style>
