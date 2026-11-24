<!--
	DayRow — Runhappy design system
	Figma: UI Library | Runhappy → Components → day-row
	https://www.figma.com/design/Ax9De11Plc2yPyfQHrGuJn/UI-Library-%7C-Runhappy?node-id=233-751

	Opening hours for one day (Self Onboarding): Switch with the day name, then "from" TimeField, "to", "to"
	TimeField. Closed shows "Closed" instead of the times. The day name has a fixed width (96) so the "to" column
	lines up from row to row (staging does not). Padding 8/16, gap 12.
	Turning a day off or on (Adriana, 2026-11-24): the times fade out while the row closes smoothly, and "Closed"
	fades in in the same place (250 ms, ease in-out). Both share one grid cell, so nothing jumps. No motion with reduced motion.
-->
<script lang="ts" module>
	export interface DayRowProps {
		/** Day name, e.g. "Monday". Also the switch label. */
		day: string;
		/** Open that day. Use `bind:open` to read it. */
		open?: boolean;
		/** Opening time, "HH:MM" (24 h). Use `bind:from`. */
		from?: string;
		/** Closing time, "HH:MM" (24 h). Use `bind:to`. */
		to?: string;
		/** Layout classes (width, margin). */
		class?: string;
	}
</script>

<script lang="ts">
	import { Switch } from '#components/Switch';
	import { TimeField } from '#components/TimeField';
	import { cubicInOut } from 'svelte/easing';
	import { prefersReducedMotion } from 'svelte/motion';
	import { fade, slide, type TransitionConfig } from 'svelte/transition';

	let {
		day,
		open = $bindable(true),
		from = $bindable('10:00'),
		to = $bindable('18:00'),
		class: className
	}: DayRowProps = $props();

	const duration = $derived(prefersReducedMotion.current ? 0 : 250);

	/** The times: fade out while the row's height closes (and the reverse when the day is turned on). */
	function collapse(node: Element): TransitionConfig {
		const base = slide(node, { duration, easing: cubicInOut });
		return { ...base, css: (t, u) => `${base.css?.(t, u) ?? ''} opacity: ${t};` };
	}
</script>

<div class={['flex items-center gap-12 px-16 py-8', className]}>
	<Switch label={day} bind:checked={open} labelClass="w-(--size-day-label)" />
	<div class="grid items-center">
		{#if open}
			<div class="col-start-1 row-start-1 flex items-center gap-12" transition:collapse>
				<TimeField
					label={`${day} opens at`}
					hideLabel
					bind:value={from}
					class="w-(--size-time-field)"
				/>
				<span class="type-ui-body-4-regular text-ui-ui-elements-muted-foreground" aria-hidden="true"
					>to</span
				>
				<TimeField
					label={`${day} closes at`}
					hideLabel
					bind:value={to}
					class="w-(--size-time-field)"
				/>
			</div>
		{:else}
			<span
				class="col-start-1 row-start-1 type-ui-body-3-regular text-ui-ui-elements-muted-foreground"
				transition:fade={{ duration }}>Closed</span
			>
		{/if}
	</div>
</div>
