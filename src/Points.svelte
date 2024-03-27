<script>
	import { onMount } from "svelte";
	import reglWrapper from "regl";
	import reglTween from "regl-tween";

	let el, regl, tween, pointsBuffer;

	export let step = 0;
	export let counts = [1, 100, 10000, 1000000];
	export let initialCoords = [0.5, 0.5];
	export let pointColor = [255, 76, 0];
	export let pointOpacity = 0.6;

	const random = () => Math.acos(-((Math.random() * 2) - 1)) / Math.PI;
	// const getRandomPositions = (count) => Array(count).fill(null).map(i => [(random() * 2) - 1, (random() * 2)  - 1]);
	const getOpacity = (step, i) => {
		if (i < counts[step]) return pointOpacity;
		return 0;
	};
	
	const positions = initialCoords ? [[initialCoords[0] / Math.pow(10, counts.length - 1), initialCoords[0] / Math.pow(10, counts.length - 1)]] : [];
	for (let i = 0; i < counts.length; i ++) {
		while (positions.length < counts[i]) {
			positions.push([Math.abs((random() * 2) - 1) / Math.pow(10, counts.length - 1 - i), Math.abs((random() * 2) - 1) / Math.pow(10, counts.length - 1 - i)]);
		}
  }
	console.log(positions)

	const setPoints = (step) => {
    const scale = Math.pow(10, counts.length - 1 - step % counts.length) * 2;
		console.log("scale", scale)
    return positions.map((p, i) => [p[0] * scale - 1, p[1] * scale - 1, getOpacity(step, i), Math.sqrt(scale) * 4]);
  };
	
	onMount(() => {
		regl = reglWrapper(el);
		tween = reglTween(regl);
	
		pointsBuffer = tween.buffer(setPoints(step), { duration: 2000 });
		
		const drawParticles = tween({
		  vert: `
		precision mediump float;
		attribute vec4 points;
		varying float opacity;
		
		void main() {
		  gl_PointSize = points[3];
		  gl_Position = vec4(points[0], points[1], 0, 1);
			opacity = points[2];
		}`,
		  frag: `
		precision lowp float;
		varying float opacity;
		uniform vec3 pointColor;
		
		void main() {
		  if (length(gl_PointCoord.xy - 0.5) > 0.5) {
		    discard;
		  }
		  gl_FragColor = vec4(pointColor, opacity);
		}`,
		
		  attributes: {
		    points: pointsBuffer,
		  },
		
		  uniforms: {
		    pointSize: 6,
				pointColor: pointColor.map(val => val / 255)
		  },

			blend: {
					enable: true,
					func: {
							srcRGB: "src alpha",
							srcAlpha: "src alpha",
							dstRGB: "one minus src alpha",
							dstAlpha: "one minus src alpha",
					},
			},

			depth: { enable: false },
		
		  count: positions.length,
		  primitive: 'points'
		});
		
		regl.frame(() => {
		  drawParticles();
		});
	});
	
	const updateBuffer = (step) => { if (pointsBuffer) pointsBuffer.update(setPoints(step)) };
	$: updateBuffer(step);
</script>

<div class="canvas-container" bind:this={el}/>

<style>
	.canvas-container {
		height: 100vh;
	}
</style>