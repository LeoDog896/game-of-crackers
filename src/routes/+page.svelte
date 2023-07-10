<script lang="ts">
	import { Canvas, Layer, type Render } from 'svelte-canvas';

	let width = 640;
	let height = 640;

	let cursorX = 0;
	let cursorY = 0;

    let isRotating = false;
    let currentRotation = 0;

	interface Vector {
		x: number;
		y: number;
	}

	let crackers: [position: Vector, rotation: number][] = [];

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
        context.fillStyle = '#4E878C';
        for (const cracker of crackers) {
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
            currentRotation += 0.1;
            if (currentRotation >= 2 * Math.PI) {
                currentRotation = 0;
                isRotating = false;
            }
        }

		// draw a rectangle at the cursor of unit size 1
        context.save();
		context.beginPath();
		context.fillStyle = 'rgba(0, 0, 0, 0.5)';
        context.translate(cursorX, cursorY);
        context.rotate(currentRotation);
        context.rect(-trueSize.x / 2, -trueSize.y / 2, trueSize.x, trueSize.y);
		context.fill();
        context.restore();
	};
</script>

<svelte:window
    bind:innerWidth={width}
    bind:innerHeight={height}
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

<Canvas
	{width}
	{height}
	on:mousemove={({ clientX, clientY }) => {
		cursorX = clientX;
		cursorY = clientY;
	}}
    on:click={() => {
        crackers.push([{ x: cursorX, y: cursorY }, currentRotation]);
    }}
>
	<Layer {render} />
</Canvas>
