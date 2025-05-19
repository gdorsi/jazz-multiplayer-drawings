<script lang="ts">
	import { onMount } from 'svelte';
	import paper from 'paper';
	import { Counter, PaintingPaths, Path, Point, Segment, Segments } from '$lib/schema';
	import { CoState } from 'jazz-svelte';
	import { Group } from 'jazz-tools';
	import { ColorPicker } from '@fuxui/colors';
	import { Button, Popover, SliderNumber, Subheading, toast } from '@fuxui/base';
	import { painting as paintingStore } from './store.svelte';
	import { base } from '$app/paths';

	let { painting }: { painting: CoState<Counter> } = $props();

	let tool: paper.Tool | null = null;

	let path: paper.Path | null = null;

	let scope: paper.PaperScope | null = null;

	let drawnPaths: { [key: string]: paper.Path } = $state({});

	let currentColor = $state('#ec4899');

	let subscription = $state(null);
	$effect(() => {
		if (subscription || !painting.current?.painting) return;

		PaintingPaths.subscribe(
			painting.current.painting.id,
			{
				resolve: {
					$each: {
						segments: {
							$each: {
								point: true,
								handleIn: true,
								handleOut: true
							}
						}
					}
				}
			},
			(painting) => {
				// get all paths
				for (let path of painting ?? []) {
					if (!path || drawnPaths[path.id]) continue;

					drawnPaths[path.id] = new paper.Path();
					drawnPaths[path.id].strokeColor = path.strokeColor as unknown as paper.Color;
					drawnPaths[path.id].strokeWidth = path.strokeWidth;

					for (let segment of path.segments ?? []) {
						if (!segment) continue;

						scope?.activate();

						let segmentPoint = new paper.Segment({
							point: new paper.Point(segment.point.x, segment.point.y),
							handleIn: new paper.Point(segment.handleIn.x, segment.handleIn.y),
							handleOut: new paper.Point(segment.handleOut.x, segment.handleOut.y)
						});

						drawnPaths[path.id].add(segmentPoint);
					}
				}
			}
		);
	});

	onMount(async () => {
		if (!canvas) {
			return;
		}

		scope = new paper.PaperScope();
		scope.setup(canvas);
		tool = new paper.Tool();

		tool.onMouseDown = (event: paper.ToolEvent) => {
			if (event.modifiers.shift) {
				return;
			}

			scope?.activate();

			path = new paper.Path();
			path.strokeColor = currentColor as unknown as paper.Color;
			path.strokeWidth = strokeWidth;

			path.add(event.point);
		};

		tool.onMouseDrag = (event: paper.ToolEvent) => {
			if (event.modifiers.shift) {
				scope?.view.translate(event.delta);
				return;
			}

			if (path === null) {
				return;
			}
			scope?.activate();

			path.add(event.point);
		};

		tool.onMouseUp = (event: paper.ToolEvent) => {
			if (event.modifiers.shift || path === null) {
				return;
			}

			scope?.activate();

			path.simplify(10);

			const group = Group.create();
			group.addMember('everyone', 'writer');

			// add path to painting
			let coSegments = Segments.create([], group);
			let coPath = Path.create(
				{
					segments: coSegments,
					strokeColor: currentColor,
					strokeWidth: strokeWidth
				},
				group
			);
			for (let segment of path.segments) {
				const position = Point.create(
					{
						x: segment.point.x,
						y: segment.point.y
					},
					group
				);
				const handleIn = Point.create(
					{
						x: segment.handleIn.x,
						y: segment.handleIn.y
					},
					group
				);
				const handleOut = Point.create(
					{
						x: segment.handleOut.x,
						y: segment.handleOut.y
					},
					group
				);
				coSegments.push(
					Segment.create(
						{
							point: position,
							handleIn: handleIn,
							handleOut: handleOut
						},
						group
					)
				);
			}

			drawnPaths[coPath.id] = path;

			painting.current?.painting?.push(coPath);
			if (painting.current) {
				painting.current.count += 1;
			}

			path = null;
		};
	});

	let canvas: HTMLCanvasElement | null = null;

	let color = $state({
		r: 0.92,
		g: 0.28,
		b: 0.6
	});

	let strokeWidth = $state(3);
</script>

<div class="fixed right-2 bottom-2 z-20 flex flex-col gap-3">
	<Popover side="left">
		{#snippet child({ props })}
			<button
				{...props}
				class="bg-base-400/30 border-base-700/30 dark:border-base-400 mt-2 flex size-8 cursor-pointer items-center justify-center overflow-hidden rounded-full border shadow-md backdrop-blur-sm"
			>
				<div
					class="rounded-full"
					style="width: {strokeWidth * 7 + 30}%; height: {strokeWidth * 7 +
						30}%; background-color: rgb({color.r * 255}, {color.g * 255}, {color.b * 255})"
				></div>
			</button>
		{/snippet}
		<Subheading class="mb-2 text-sm sm:text-base">Stroke Width</Subheading>

		<SliderNumber
			type="single"
			bind:value={strokeWidth}
			min={1}
			max={10}
			step={1}
			class="min-w-36"
		/>

		<Subheading class="mt-6 mb-2 text-sm sm:text-base">Color</Subheading>

		<ColorPicker
			bind:rgb={color}
			onchange={(color) => {
				console.log(color);
				currentColor = color.hex;
			}}
			quickSelects={[
				{
					label: 'white',
					rgb: {
						r: 1,
						g: 1,
						b: 1
					}
				},
				{
					label: 'black',
					rgb: {
						r: 0,
						g: 0,
						b: 0
					}
				},
				{
					label: 'red',
					rgb: {
						r: 0.93,
						g: 0.26,
						b: 0.26
					}
				},
				{
					label: 'blue',
					rgb: {
						r: 0.23,
						g: 0.5,
						b: 0.96
					}
				},
				{
					label: 'green',
					rgb: {
						r: 0.13,
						g: 0.77,
						b: 0.36
					}
				},
				{
					label: 'yellow',
					rgb: {
						r: 0.91,
						g: 0.7,
						b: 0.03
					}
				},
				{
					label: 'pink',
					rgb: {
						r: 0.92,
						g: 0.28,
						b: 0.6
					}
				}
			]}
		/>
	</Popover>
</div>
<canvas bind:this={canvas} class="fixed h-screen w-screen"></canvas>

<div class="fixed top-2 left-2 z-20">
	<Button href={base + '/'}>Back</Button>
	<Button
		onclick={() => {
			toast('Link copied to clipboard', {
				description: 'Send this link to your friends to draw together'
			});
			navigator.clipboard.writeText(window.location.href);
		}}
	>
		Share
	</Button>
</div>
