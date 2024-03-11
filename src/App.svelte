<script>
	import { Hero, Scroller, ScrollerSection } from "@onsvisual/svelte-components";
	import Points from "./Points.svelte";

	let step = 0;
</script>

<Hero title="Zooming points demo" background="#555">
	<p style:font-size="larger">Scroll down to view the animation</p>
</Hero>

<Scroller bind:index={step} marginBottom={false}>
	<div slot="background">
		<Points {step}/>
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
