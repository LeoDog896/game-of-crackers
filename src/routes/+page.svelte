<script lang="ts">
	import { Canvas, Layer, type Render } from 'svelte-canvas';
	import intersect from "polygons-intersect"

	let containerWidth = 0;
	let containerHeight = 0;

	$: width = Math.min(containerWidth, containerHeight);
	$: height = Math.min(containerWidth, containerHeight);

	enum Player {
		P1,
		P2,
	}

	let currentPlayer: Player = Player.P1;

	let cursorX = 0;
	let cursorY = 0;

    let isRotating = false;
    let currentRotation = 0;
	let rotationDisabled = false;

	type Cracker = [position: Vector, rotation: number, player: Player];

	function corners(cracker: Cracker, size: Vector): Vector[] {
		const [position, rotation] = cracker;
		const { x: w, y: h } = size;

		const cos = Math.cos(rotation);
		const sin = Math.sin(rotation);

		const corners: Vector[] = [
			{ x: -w / 2, y: -h / 2 },
			{ x: w / 2, y: -h / 2 },
			{ x: w / 2, y: h / 2 },
			{ x: -w / 2, y: h / 2 }
		]

		const rotatedCorners = corners.map(({ x, y }) => ({
			x: x * cos - y * sin,
			y: x * sin + y * cos
		}));

		return rotatedCorners.map(({ x, y }) => ({
			x: x + position.x,
			y: y + position.y
		}));
	}
	
	function isColliding(size: Vector, crackerA: Cracker, crackerB: Cracker): boolean {
		return intersect((corners(crackerA, size)), (corners(crackerB, size))).length > 0;
	}

	interface Vector {
		x: number;
		y: number;
	}

	let crackers: Cracker[] = [];

	let cellDimensions: Vector = { x: 4, y: 4 };

	let render: Render;
	$: render = ({ context, width, height }) => {
		const trueSize: Vector = {
			x: width / cellDimensions.x,
			y: height / cellDimensions.y
		};

		// draw a grid
		context.strokeStyle = '#ccc';
		context.lineWidth = 1;
		for (let x = 0; x < cellDimensions.x; x++) {
			context.beginPath();
			context.moveTo(x * trueSize.x, 0);
			context.lineTo(x * trueSize.x, height);
			context.stroke();
		}

		for (let y = 0; y < cellDimensions.y; y++) {
			context.beginPath();
			context.moveTo(0, y * trueSize.y);
			context.lineTo(width, y * trueSize.y);
			context.stroke();
		}

        // draw the crackers
        for (const cracker of crackers) {
			if (cracker[2] == Player.P1) {
				context.fillStyle = '#5BC0EB';
			} else {
				context.fillStyle = '#9BC53D';
			}
            context.save();
            context.beginPath();
            context.translate(cracker[0].x, cracker[0].y);
            context.rotate(cracker[1]);
            context.rect(-trueSize.x / 2, -trueSize.y / 2, trueSize.x, trueSize.y);
            context.fill();
            context.restore();
        }

        // handle rotation
        if (isRotating) {
            currentRotation += 2 * Math.PI / 60;
            if (currentRotation >= 2 * Math.PI) {
                currentRotation = 0;
                isRotating = false;
            }
        }

		// draw a rectangle at the cursor of unit size 1
        context.save();
		context.beginPath();
		let collidingCrackers = [];

		for (const cracker of crackers) {
			if (isColliding(trueSize, cracker, [{ x: cursorX, y: cursorY }, currentRotation, 0])) {
				collidingCrackers.push(cracker);
			}
		}

		const onEdge = corners([{ x: cursorX, y: cursorY }, currentRotation, 0], trueSize).some(({ x, y }) => {
			return x < 0 || x > width || y < 0 || y > height;
		});

		if (collidingCrackers.length > 0 || onEdge) {
			context.fillStyle = "rgba(195, 66, 63, 0.5)";
			rotationDisabled = true;
		} else {
			context.fillStyle = 'rgba(0, 0, 0, 0.5)';
			rotationDisabled = false;
		}

        context.translate(cursorX, cursorY);
        context.rotate(currentRotation);
        context.rect(-trueSize.x / 2, -trueSize.y / 2, trueSize.x, trueSize.y);
		context.fill();
        context.restore();
	};
</script>

<svelte:window
    on:keydown={({ key }) => {
        if (key === 'r') {
            isRotating = true;
        }
    }}
    on:keyup={({ key }) => {
        if (key === 'r') {
            isRotating = false;
        }
    }}
/>

<main>
	<h1>Game of Crackers</h1>

	<div class="container" bind:clientWidth={containerWidth} bind:clientHeight={containerHeight}>
		<Canvas
			{width}
			{height}
			style="border: 1px solid #ccc; {rotationDisabled ? 'cursor: not-allowed' : ''}"
			on:mousemove={({ offsetX, offsetY }) => {
				cursorX = offsetX;
				cursorY = offsetY;
			}}
			on:click={() => {
				if (rotationDisabled) return;
				crackers = [...crackers, [{ x: cursorX, y: cursorY }, currentRotation, currentPlayer]];
				currentPlayer = currentPlayer === Player.P1 ? Player.P2 : Player.P1;
			}}
		>
			<Layer {render} />
		</Canvas>
	</div>

	<p>Click to move. Press <code>r</code> to rotate. Last player to be able to move wins.</p>
	</main>

<style>
	.container {
		width: 100%;
		height: 80%;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	h1 {
		margin-top: 0;
	}

	main {
		height: 100%;
		width: 100%;
		text-align: center;
	}
</style>