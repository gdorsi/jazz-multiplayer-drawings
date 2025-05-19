<script lang="ts">
	import { onMount } from 'svelte';
	import paper from 'paper';
	import { Counter, PaintingPaths } from '$lib/schema';

	let { painting }: { painting: Counter } = $props();

	let drawnPaths: Set<string> = $state(new Set());
	let subscription = $state(false);
	let scope: paper.PaperScope | null = null;
	let canvas: HTMLCanvasElement | null = null;

	onMount(async () => {
		if (!canvas) {
			return;
		}
		scope = new paper.PaperScope();

		scope.setup(canvas);
	});

	function viewFitBounds(itemBounds: paper.Rectangle) {
		if (!scope) return;

		var viewBounds = scope.view.bounds;
		var scaleRatio = Math.min(
			viewBounds.width / itemBounds.width,
			viewBounds.height / itemBounds.height
		);
		scope.view.translate({
			x: viewBounds.center.x - itemBounds.center.x,
			y: viewBounds.center.y - itemBounds.center.y
		});
		scope.view.scale(scaleRatio);
	}

	function fitAll() {
		if (!scope) return;

		const layer = scope.project.activeLayer;
		if (layer.children.length === 0) return;
		// Optionally give a little padding
		const padding = 20;
		const bounds = layer.bounds.expand(padding);
		viewFitBounds(bounds);
	}

	$effect(() => {
		if (subscription || !painting?.painting) return;

		subscription = true;

		PaintingPaths.subscribe(
			painting.painting.id,
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
				for (let path of painting ?? []) {
					if (!path || drawnPaths.has(path.id)) continue;

					scope?.activate();
					drawnPaths.add(path.id);

					let drawPath = new paper.Path();
					drawPath.strokeColor = path.strokeColor as unknown as paper.Color;
					drawPath.strokeWidth = path.strokeWidth;

					for (let segment of path.segments ?? []) {
						if (!segment) continue;

						scope?.activate();

						let segmentPoint = new paper.Segment({
							point: new paper.Point(segment.point.x, segment.point.y),
							handleIn: new paper.Point(segment.handleIn.x, segment.handleIn.y),
							handleOut: new paper.Point(segment.handleOut.x, segment.handleOut.y)
						});

						drawPath.add(segmentPoint);

						fitAll();
					}
				}
			}
		);
	});
</script>

<canvas bind:this={canvas} class="h-full w-full"></canvas>
