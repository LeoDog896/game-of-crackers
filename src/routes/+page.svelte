<script lang="ts">
	import { Canvas, Layer, type Render } from 'svelte-canvas';

	let width = 640;
	let height = 640;

	let cursorX = 0;
	let cursorY = 0;

    let isRotating = false;
    let currentRotation = 0;
	let rotationDisabled = false;

	type Cracker = [position: Vector, rotation: number]

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
		// check if 2 crackers are colliding independent of the mouse
		for (const cracker of crackers) {
			const points = corners(cracker, size);
			for (let i1 = 0; i1 < points.length; i1++) {
				const i2 = (i1 + 1) % points.length;
				const p1 = points[i1];
				const p2 = points[i2];

				const normal = { x: p2.y - p1.y, y: p1.x - p2.x };

				let minA = null;
				let maxA = null;
				for (const p of corners(crackerA, size)) {
					const projected = normal.x * p.x + normal.y * p.y;
					if (minA == null || projected < minA) minA = projected;
					if (maxA == null || projected > maxA) maxA = projected;
				}

				let minB = null;
				let maxB = null;
				for (const p of corners(crackerB, size)) {
					const projected = normal.x * p.x + normal.y * p.y;
					if (minB == null || projected < minB) minB = projected;
					if (maxB == null || projected > maxB) maxB = projected;
				}

				if ((maxA && maxB && minA && minB) && (maxA < minB || maxB < minA)) return false;
			}
		}

		return true;
	}

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
			if (isColliding(trueSize, cracker, [{ x: cursorX, y: cursorY }, currentRotation])) {
				collidingCrackers.push(cracker);
			}
		}

		if (collidingCrackers.length > 0) {
			context.fillStyle = "rgba(255, 0, 0, 0.5)";
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
	style="{rotationDisabled ? 'cursor: not-allowed' : ''}"
	on:mousemove={({ clientX, clientY }) => {
		cursorX = clientX;
		cursorY = clientY;
	}}
    on:click={() => {
		if (rotationDisabled) return;
        crackers = [...crackers, [{ x: cursorX, y: cursorY }, currentRotation]];
    }}
>
	<Layer {render} />
</Canvas>
