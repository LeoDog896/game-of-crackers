<script lang="ts">
	import { Canvas, Layer, type Render } from 'svelte-canvas';

	let width = 640;
	let height = 640;

	let cursorX = 0;
	let cursorY = 0;

	interface Vector {
		x: number;
		y: number;
	}

	let crackers: Vector[] = [];

	let cellDimensions: Vector = { x: 10, y: 10 };

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
        context.fillStyle = '#f00';
        for (const cracker of crackers) {
            context.beginPath();
            context.rect(cracker.x - trueSize.x / 2, cracker.y - trueSize.y / 2, trueSize.x, trueSize.y);
            context.fill();
        }

		// draw a rectangle at the cursor of unit size 1
		context.beginPath();
		context.fillStyle = '#000';
		context.rect(cursorX - trueSize.x / 2, cursorY - trueSize.y / 2, trueSize.x, trueSize.y);
		context.fill();
	};
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />

<Canvas
	{width}
	{height}
	on:mousemove={({ clientX, clientY }) => {
		cursorX = clientX;
		cursorY = clientY;
	}}
    on:click={() => {
        crackers.push({ x: cursorX, y: cursorY });
    }}
>
	<Layer {render} />
</Canvas>
