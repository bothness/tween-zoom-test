<script>
	import { Theme, Hero, Scroller, ScrollerSection } from "@onsvisual/svelte-components";
	import Points from "./Points.svelte";

	let step = 0;
</script>

<Theme theme="dark">
	<Hero title="Zooming points demo" background="#555">
		<p style:font-size="larger">Scroll down to view the animation</p>
	</Hero>
	<Scroller theme="dark" bind:index={step} marginBottom={false}>
		<div slot="background">
			<Points
				{step}
				counts={[1, 100, 10000, 1000000]}
				initialCoords={[0.5, 0.5]}
				pointColor={[255, 76, 0]}
				pointOpacity={0.5}/>
		</div>
		<div slot="foreground">
			<ScrollerSection>
				One point.
			</ScrollerSection>
			<ScrollerSection>
				 One hundred points.
			</ScrollerSection>
			<ScrollerSection>
				 Ten thousand points.
			</ScrollerSection>
			<ScrollerSection>
				 One million points.
			</ScrollerSection>
		</div>
	</Scroller>
</Theme>
