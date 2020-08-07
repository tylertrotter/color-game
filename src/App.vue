<template>
  <div id='app'>
		<canvas 
			ref='canvas'
			:width='squareSize * width * scale' 
			:height='squareSize * height * scale' 
			@click='colorSquare(playerOne)'
		>
		</canvas>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {
    
	},
	data() {
		return {
			scale: 3,
			squareSize: 10,
			width: 80,
			height: 60,
			environment: [],
			playerOne: {
					originSquare: [0, 0],
					position: [7, 2],
					color: 200,
					fertility: 1,
					mutationLevel: 2 
			},
			players: [
				{
					originSquare: [0, 0],
					position: [3, 10],
					color: 300,
					movement: 'bishop', // rook, bishop, queen
					speed: 2, // 1,2,3
					direction: 'e',
					fertility: 1,
					mutationLevel: 2 
				},
			]
		}
	},
	computed: {
		ctx(){
			return this.$refs.canvas.getContext('2d');
		}
	},
	mounted() {
		this.generateEnvironment();
		this.paintBoard();
		
		document.addEventListener('keydown', e => { this.handleKeyUp(e) });
	},
	methods: {
		handleKeyUp(e) {
			e.preventDefault();
			e.stopPropagation();
			if(e.key === 's' || e.key === ' ')
				this.colorSquare(this.playerOne);
			else
				this.move(e);
		},
		generateEnvironment() {
			this.ctx.scale(this.scale, this.scale);

			for(let i = 0; i < this.width * this.height; i++){
				if(Math.floor(Math.random() * 50) === 15)
					this.environment.push([null, 0]);
				else
					this.environment.push([null, Math.random() * 2 + 5]);
			}
			this.paintBoard();
		},
		paintBoard() {
			this.environment.forEach((square, i) => {
				this.ctx.fillStyle = `hsl(${square[0] || 0}, ${square[0] ? 100 : 0}%, ${square[1]*10}%)`;
				this.ctx.fillRect(Math.floor(i % this.width) * this.squareSize, Math.floor(i / this.width) * this.squareSize, this.squareSize, this.squareSize);
			});

			this.placePlayer(this.playerOne);

			this.players.forEach(player => {
				this.placePlayer(player);
			});

		},
		placePlayer(player) {
			// direction is a proxy for not playerOne
			const playerSize = player.direction ? .25 : .4;
			this.ctx.lineWidth = .75;
			this.ctx.fillStyle = `hsl(${player.color}, 100%, 50%)`;
			this.ctx.beginPath();
			this.ctx.arc(player.position[0] * this.squareSize + 5, this.squareSize * (player.position[1] + 1) - (this.squareSize/2), this.squareSize * playerSize, 0, Math.PI * 2, true);
			this.ctx.fill();
			this.ctx.strokeStyle = `white`;
			
			this.ctx.stroke();
		},
		colorSquare(player){
			const i = this.width * player.position[1] + player.position[0];
			this.environment[i][0] = player.color;

			this.paintBoard();
			this.placePlayer(player);

			this.players.forEach(player => {
				this.automaticMove(player);
			});
		},
		getSquare(x, y) {
			return this.environment[this.width * y + x];
		},
		move(e){
			const position = this.playerOne.position;
			const x = this.playerOne.position[0];
			const y = this.playerOne.position[1];

			if(e.key === 'w' && y > 0 && this.getSquare(x, y - 1)[1] > 0) {
				position[1] = y - 1;
			}else if(e.key === 'e' && y > 0 && x < this.width-1) {
				position[1] = y - 1;
				position[0] = x + 1;
			}else if(e.key === 'd')
				position[0] = x + 1;
			else if(e.key === 'c') {
				position[1] = y + 1;
				position[0] = x + 1;
			}else if(e.key === 'x')
				position[1] = y + 1;
			else if(e.key === 'z' && x > 0) {
				position[1] = y + 1;
				position[0] = x - 1;
			}else if(e.key === 'a' && x > 0)
				position[0] = x - 1;
			else if(e.key === 'q' && x > 0  && y > 0) {
				position[1] = y - 1;
				position[0] = x - 1;
			}else {
				return;
			}

			this.players.forEach(player => {
				this.automaticMove(player);
			});

		},
		getNewDirection(movement, oldDirection) {
			const directions = {
				queen: ['q', 'w', 'e', 'a', 'd', 'z', 'x', 'c'],
				bishop: ['q', 'e', 'z', 'c'],
				rook: ['w', 'a', 'd', 'x']
			}
			directions[movement].splice(directions[movement].indexOf(oldDirection), 1);
			return directions[movement][Math.floor(Math.random() * directions[movement].length)];

		},
		automaticMove(player) {
			const position = player.position;
			const direction = player.direction;
			const speed = player.speed;
			const x = player.position[0];
			const y = player.position[1];

			if(direction === 'w' && y >= speed && this.pathIsClear([x, y], [0, -1], speed))
				position[1] = y - speed;
			else if(direction === 'e' && y >= speed && this.pathIsClear([x, y], [+1, -1], speed)) {
				position[0] = x + speed;
				position[1] = y - speed;
			}else if(direction === 'd' && this.pathIsClear([x, y], [+1, 0], speed))
				position[0] = x + speed;
			else if(direction === 'c' && this.pathIsClear([x, y], [+1, +1], speed)) {
				position[0] = x + speed;
				position[1] = y + speed;
			}else if(direction === 'x' && this.pathIsClear([x, y], [0, -1], speed))
				position[1] = y + 1 * speed;
			else if(direction === 'z' && x >= speed  && this.pathIsClear([x, y], [-1, +1], speed)) {
				position[0] = x - speed;
				position[1] = y + speed;
			}else if(direction === 'a' && x >= speed && this.pathIsClear([x, y], [-1, 0], speed))
				position[0] = x - speed;
			else if(direction === 'q' && x >= speed && this.pathIsClear([x, y], [-1, -1], speed)) {
				position[0] = x - speed;
				position[1] = y - speed;
			}else{
				player.direction = this.getNewDirection(player.movement, direction);
				this.automaticMove(player);
			}

			this.paintBoard();
		},
		pathIsClear(origin, direction, speed) {
			for (let i = speed; i > 0; i--) {
				if(this.getSquare(origin[0] + direction[0] * i, origin[1] + direction[1] * i)[1] === 0)
					return false;
			}
			return true;
		}
	}
}
</script>

<style>
	body {
		margin: 0;
	}
</style>
