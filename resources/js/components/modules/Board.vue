<template>
<div class="game_board">

	<svg width="100%" height="100%" id="board24" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 1170 1170"
		:class="'player_' + player.color">
		<title>castle-board</title>
		<!-- START WALL HOLES -->
		<ellipse
			v-for="(pinHole, index) in boardData.coords"
			v-if="pinHole.fake == null" class="deck-card"
			v-bind:id="'pinhole_'+ index"
			v-bind:class="[
				'pinhole', 'cls-' + index,
				{ 'active': board.pins[index] != null && board.pins[index].pin != null },
				{ 'route': board.moveOptions[index] != null && board.moveOptions[index].route != null },
				{ 'move': board.moveOptions[index] != null && board.moveOptions[index].move != null },
				{ 'focused': board.focusedPin != null && board.focusedPin == index },
				{ 'wall': board.holdingWalls[index] != null && board.holdingWalls[index] },
				board.pins[index] != null ? board.pins[index].pin.color : '',
				pinHole.customClass
			]"
			v-bind:cx="pinHole.cx" v-bind:cy="pinHole.cy" v-bind:rx="pinHole.rx" v-bind:ry="pinHole.ry"
			v-on:click="pinHoleClicked($event)"/>
	</svg>

	<div class="move_info">
		<span class="">{{ deck.picked != null ? 'Card: ' + deck.picked.value : '' }}</span>
		<span class="">{{ board.currentStepInfo }}</span>
	</div>

</div>
</template>

<script>
export default {
	props: ['player', 'boardData', 'board', 'deck', 'helpers'],
	data () {
		return {
			//
		}
	},
	mounted() {
		console.log('#CastleBoard Mounted');
	},
	methods: {
		pinHoleClicked: function(event) {
			let self = this;
			let pinHole = $(event.target);

			if(pinHole.hasClass('move')){
				self.runMove(pinHole);
			} else if (pinHole.hasClass('focused')) {
				self.board.focusedPin = null;
				self.board.moveOptions = {};
			} else if (pinHole.hasClass('active')) {

				if(pinHole.hasClass(self.player.color) || pinHole.hasClass(self.player.color[1])) {
					if (pinHole.hasClass('home_pinhole')) {
					} else {
						self.showPossibleMoves(pinHole);
					}
				} else {
				}
			} else {
				self.board.focusedPin = null;
				self.board.moveOptions = {};
			}
		},
		showPossibleMoves: function(pinHole) {
			let self = this;
			let startPoss = parseInt(pinHole.attr('id').replace('pinhole_',''));
			self.board.moveOptions = {};
			if(self.deck.picked != null) {
				let value = self.deck.picked.value;
				switch (value) {
					case "B":
						if(!self.board.holdingWalls[startPoss]) {
							self.showJackMove(startPoss);
						}
						break;

					default:
						self.createPossibleMoves(startPoss);
				}
			}
		},
		createPossibleMoves: function(startPoss) {

			let self = this;
			let cardData = self.deck.picked;
			let isSeven = cardData.value == 7 ? true : false;

			if (cardData) {

				let cardMoves = self.deck.picked.moves;
				// let startPoss = parseInt(pinHole.attr('id').replace('pinhole_',''));

				self.board.focusedPin = startPoss;

				let moves = {};
				let route = {};

				let isBlockedByWall = false;

				let kills = [];

				let homeMoves = {};
				let homeEntrances = {
					red: 60,
					green: 15,
					orange: 30,
					blue: 45
				};

				let pinHomeEntrance = homeEntrances[self.board.pins[startPoss].pin.color];

				for (let moveIndex = 0; moveIndex < cardMoves.length; moveIndex++) {
					let cardMove = cardMoves[moveIndex];
					let pinHoleIndex = startPoss + cardMove <= 0 ? (60 + (startPoss+cardMove)) : startPoss+cardMove > 60 ? (startPoss + cardMove) - 60 : startPoss+cardMove;

					if (cardMove > 0) {
						for (let routeIndex = 1; routeIndex <= cardMove; routeIndex++) {
							let currentPinHoleIndex = startPoss+routeIndex <= 0 ? (60 + (startPoss+routeIndex)) : startPoss+routeIndex > 60 ? (startPoss + routeIndex) - 60 : startPoss+routeIndex;
							let modifiedPinHoleIndex = currentPinHoleIndex-1 == 0 ? 60 : currentPinHoleIndex-1;
							if (modifiedPinHoleIndex === pinHomeEntrance) {
								console.log("with modifiedPinHoleIndex");
								homeMoves = self.createPossibleHomeMoves(startPoss, (cardMove - routeIndex) + 1, isSeven, kills.slice());
							}
							if (self.board.holdingWalls[currentPinHoleIndex] != null) {
								isBlockedByWall = true;
								break;
							}
						}
					} else {
						for (let countIndex = cardMove + 1; countIndex <= 0; countIndex++) {
							let routeIndex = cardMove - countIndex;
							let currentPinHoleIndex = startPoss+routeIndex <= 0 ? (60 + (startPoss+routeIndex)) : startPoss+routeIndex > 60 ? (startPoss + routeIndex) - 60 : startPoss+routeIndex;
							if (currentPinHoleIndex === pinHomeEntrance) {
								homeMoves = self.createPossibleHomeMoves(startPoss, cardMove - routeIndex, isSeven, []);
							}
							if (self.board.holdingWalls[currentPinHoleIndex] != null) {
								isBlockedByWall = true;
								break;
							}
						}
					}

					if (isBlockedByWall) {
						isBlockedByWall = false;
						continue;
					}

					let move = {
						move: true,
						from: startPoss
					};

					if(!jQuery.isEmptyObject(homeMoves)) {
						moves = Object.assign(moves, homeMoves);
					}
					if (!isSeven || self.board.pins[pinHoleIndex] == null) {
						if (kills.length > 0) {
							move.kills = kills.slice();
						}
						moves[pinHoleIndex] = move;
					} else {
						kills.push(pinHoleIndex);
					}
				}

				self.board.moveOptions = moves;
			}
		},
		createPossibleHomeMoves: function(startPoss, moveCount, isSeven, kills) {
			let self = this;
			moveCount =  -moveCount > 0 ? -moveCount : moveCount;
			console.log("home init", moveCount, kills);
			if (self.board.holdingWalls[startPoss] != null) {
				return {};
			}


			let moves = {};
			let homeStarts = {
				red: 61,
				green: 65,
				orange: 69,
				blue: 73
			};

			let homeStart = homeStarts[self.board.pins[startPoss].pin.color];
			for (var moveIndex = 0; moveIndex < moveCount; moveIndex++) {
				let currentHomePin = homeStart + moveIndex;
				if (self.board.pins[currentHomePin] != null) {
					break;
				} else if (currentHomePin > (homeStart + 3)) {
					break;
				} else {
					if(isSeven) {
						let move = {
							move: true,
							from: startPoss
						};
						if (kills.length > 0) {
							move.kills = kills.slice();
						}
						moves[currentHomePin] = move;
					} else if (moveIndex === (moveCount - 1)) {
						let move = {
							move: true,
							from: startPoss
						};
						moves[currentHomePin] = move;
					}
				}
			}

			// moves[61] = move;
			return moves;
		},
		showJackMove: function(startPoss) {
			let self = this;

			// let startPoss = parseInt(pinHole.attr('id').replace('pinhole_',''));

			let moves = {};
			for (let key in self.board.pins) {

				if (key <= 60 && key != startPoss && self.board.holdingWalls[key] == null) {
					moves[key] = {
						from: startPoss,
						swap: true,
						move: true
					};
				}

			}
			self.board.focusedPin = startPoss;
			self.board.moveOptions = moves;
		},
		runMove: function(pinHole) {

			var self = this;
			let movePinId = parseInt(pinHole.attr('id').replace('pinhole_',''));


			let origMovePin = self.board.moveOptions[movePinId];
			let startPinId = origMovePin.from;
			let startPin = self.board.pins[startPinId];

			const defaultHomePinHoleColors = {
				1: 'red',
				16: 'green',
				31: 'orange',
				46: 'blue'
			};

			if (origMovePin.swap) {
				let movePin = {
					pin: {
						color: self.board.pins[startPinId].pin.color
					}
				};
				let swapPin = {
					pin: {
						color: self.board.pins[movePinId].pin.color
					}
				};
				self.$set(self.board.pins, movePinId, movePin);
				self.$set(self.board.pins, startPinId, swapPin);
			} else {
				let movePin = {};
				if (startPinId === 0) {
					movePin = {
						pin: {
							color: defaultHomePinHoleColors[movePinId]
						}
					};
					self.$set(self.board.holdingWalls, movePinId, true);
				} else {
					movePin = {
						pin: {
							color: self.board.pins[startPinId].pin.color
						}
					};
					if(self.board.holdingWalls[startPinId]) {
						self.$delete(self.board.holdingWalls, startPinId);
					}
					if (origMovePin.kills != null) {
						for (var killIndex = 0; killIndex < origMovePin.kills.length; killIndex++) {
							let kill = origMovePin.kills[killIndex];
							self.$delete(self.board.pins, kill);
						}
					}
					self.$delete(self.board.pins, startPinId);
				}
				self.$set(self.board.pins, movePinId, movePin);
			}
			self.cleanUpAfterMove();
		},
		cleanUpAfterMove: function() {
			let self = this;
			// CleanUp Move
			for (var cardIndex = 0; cardIndex < self.deck.cards.length; cardIndex++) {
				let card = self.deck.cards[cardIndex];
				if (card === self.deck.picked) {
					self.$delete(self.deck.cards, cardIndex);
				}
			}

			self.board.history.push($.extend(true, {}, self.board.pins));
			self.board.focusedPin = null;
			self.board.moveOptions = {};
			self.deck.picked = null;
			self.board.currentStepInfo = "Pick Card from Hand!";
			self.helpers.playerTurnFinished(self.player);
		}
	}

}
</script>

<style lang="scss">
$pin-hole-fill: #353535bf;
$pin-hole-stroke: #616161;
$pin-hole-stroke-hover: lighten($pin-hole-stroke, 10%);

$pin-red: #900b10;
$pin-green: #056733;
$pin-orange: #b94d1a;
$pin-blue: #044bb5;
$move-color: #fff;

.game_board {
	position: relative;
	// height: 80vh;
	padding: .66rem;
	min-height: 320px;
	min-width: 320px;

	svg {
		transition: transform .42s;

		&.player_red {
			transform: rotate(0deg);
		}
		&.player_green {
			transform: rotate(90deg);
		}
		&.player_orange {
			transform: rotate(180deg);
		}
		&.player_blue {
			transform: rotate(270deg);
		}
	}

	.pinhole {
		cursor: pointer;
		stroke: $pin-hole-stroke;
		fill: $pin-hole-fill;
		stroke-miterlimit: 6;
		stroke-width: 6px;

		&.cls-1, &.red_home-1, &.red_home-2, &.red_home-3, &.red_home-4 {
			stroke: $pin-red;
			&:hover {
				stroke: lighten($pin-red, 5%) !important;
			}
		}
		&.cls-16, &.green_home-1, &.green_home-2, &.green_home-3, &.green_home-4 {
			stroke: $pin-green;
			&:hover {
				stroke: lighten($pin-green, 5%) !important;
			}
		}
		&.cls-31, &.orange_home-1, &.orange_home-2, &.orange_home-3, &.orange_home-4 {
			stroke: $pin-orange;
			&:hover {
				stroke: lighten($pin-orange, 5%) !important;
			}
		}
		&.cls-46, &.blue_home-1, &.blue_home-2, &.blue_home-3, &.blue_home-4 {
			stroke: $pin-blue;
			&:hover {
				stroke: lighten($pin-blue, 5%) !important;
			}
		}

		&:hover {
			stroke: $pin-hole-stroke-hover;
		}

		&.active {

			&.red {
				// stroke: $pin-red;
				fill: $pin-red;
				&:hover {
					stroke: darken($pin-red, 5%);
				}
			}
			&.green {
				// stroke: $pin-green;
				fill: $pin-green;
				&:hover {
					stroke: darken($pin-green, 5%);
				}
			}
			&.orange {
				// stroke: $pin-orange;
				fill: $pin-orange;
				&:hover {
					stroke: darken($pin-orange, 5%);
				}
			}
			&.blue {
				// stroke: $pin-blue;
				fill: $pin-blue;
				&:hover {
					stroke: darken($pin-blue, 5%);
				}
			}

		}

		&.route {
			stroke: #855A3E;
			opacity: .6;
		}

		&.move {
			stroke: $move-color !important;
			stroke-width: 10px;
			stroke-miterlimit: 5;
		}

		&.wall {
			// stroke: #000 !important;
			stroke-width: 2px;
			stroke-miterlimit: 0;
		}

		&.focused {
			// stroke: $move-color !important;
			stroke-width: 10px;
			stroke-miterlimit: 5;

			&.red {
				stroke: darken($pin-red, 5%) !important;
				fill: darken($pin-red, 10%);
			}
			&.green {
				stroke: darken($pin-green, 5%) !important;
				fill: darken($pin-green, 10%);
			}
			&.orange {
				stroke: darken($pin-orange, 5%) !important;
				fill: darken($pin-orange, 10%);
			}
			&.blue {
				stroke: darken($pin-blue, 5%) !important;
				fill: darken($pin-blue, 10%);
			}
		}

	}

	.move_info {
		position: absolute;
		display: flex;
		flex-flow: column;
		flex-wrap: wrap;
		height: 220px;
		width: 220px;
		top: calc(50% - 110px);
		left: calc(50% - 110px);
		text-align: center;
		justify-content: center;
		font-size: 120%;
		color: #fff;

		span {
			align-self: center;
		}
	}

}
</style>
