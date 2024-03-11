<script>
	import { onMount } from "svelte";
	import reglWrapper from "regl";
	import reglTween from "regl-tween";

	let el, regl, tween, pointsBuffer;

	export let step = 0;
	
	const counts = [1, 100, 10000, 1000000];
	const random = () => Math.acos(-((Math.random() * 2) - 1)) / Math.PI;
	// const getRandomPositions = (count) => Array(count).fill(null).map(i => [(random() * 2) - 1, (random() * 2)  - 1]);
	const getOpacity = (step, i) => {
		let count = 0;
		for (let j = 0; j <= step; j ++) {
			count += counts[j];
			if (i < count) return 0.8;
		}
		return 0;
	};
	
	const positions = [];
	for (let i = 0; i < counts.length; i ++) {
    const c = counts[i];
		if (i === 0) {
			positions.push([0, 0]);
		} else {
			for (let j = 0; j < c; j ++) {
				positions.push([((random() * 2) - 1) / Math.pow(10, counts.length - 1 - i), ((random() * 2) - 1) / Math.pow(10, counts.length - 1 - i)]);
			}
		}
  }

	const setPoints = (step) => {
    const scale = Math.pow(10, counts.length - 1 - step % counts.length);
		console.log("scale", scale)
    return positions.map((p, i) => [p[0] * scale, p[1] * scale, getOpacity(step, i), Math.sqrt(scale) * 4]);
  };
	
	onMount(() => {
		regl = reglWrapper(el);
		tween = reglTween(regl);
	
		pointsBuffer = tween.buffer(setPoints(step), { duration: 2000 });
		
		const drawParticles = tween({
		  vert: `
		precision mediump float;
		attribute vec4 points;
		uniform float pointSize;
		varying float opacity;
		
		void main() {
		  gl_PointSize = points[3];
		  gl_Position = vec4(points[0], points[1], 0, 1);
			opacity = points[2];
		}`,
		  frag: `
		precision lowp float;
		varying float opacity;
		
		void main() {
		  if (length(gl_PointCoord.xy - 0.5) > 0.5) {
		    discard;
		  }
		  gl_FragColor = vec4(1.0, 0.3, 0.0, opacity);
		}`,
		
		  attributes: {
		    points: pointsBuffer,
		  },
		
		  uniforms: {
		    pointSize: 6,
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
		
		  count: counts.reduce((a, b) => a + b, 0),
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