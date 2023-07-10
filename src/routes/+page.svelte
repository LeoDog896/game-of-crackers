<script lang="ts">
	import { Canvas, Layer, type Render } from 'svelte-canvas';

    let width = 640;
    let height = 640;

    interface Vector {
        x: number;
        y: number;
    }

    let cellDimensions: Vector = { x: 10, y: 10 };

	let render: Render;
	$: render = ({ context, width, height }) => {
        // draw a grid
        context.strokeStyle = '#ccc';
        context.lineWidth = 1;
        for (let x = 0; x < cellDimensions.x; x++) {
            context.beginPath();
            context.moveTo(x * width / cellDimensions.x, 0);
            context.lineTo(x * width / cellDimensions.x, height);
            context.stroke();
        }

        for (let y = 0; y < cellDimensions.y; y++) {
            context.beginPath();
            context.moveTo(0, y * height / cellDimensions.y);
            context.lineTo(width, y * height / cellDimensions.y);
            context.stroke();
        }
	};
</script>

<svelte:window bind:innerWidth={width} bind:innerHeight={height} />

<Canvas {width} {height}>
	<Layer {render} />
</Canvas>
