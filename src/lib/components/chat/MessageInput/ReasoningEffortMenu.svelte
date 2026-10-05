<script lang="ts">
	// FORK(reasoning-effort): Reasoning effort selector for the chat input.
	// Writes `params.reasoning_effort` (the same field as Chat Controls -> Advanced Params),
	// so the value is persisted per chat and sent with every request.
	// Hidden when every selected model has the `reasoning_effort` capability disabled.
	import { getContext } from 'svelte';

	import { models } from '$lib/stores';

	import Dropdown from '$lib/components/common/Dropdown.svelte';
	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import Bolt from '$lib/components/icons/Bolt.svelte';

	const i18n = getContext('i18n');

	export let params: Record<string, any> = {};
	export let selectedModelIds: string[] = [];
	export let disabled = false;

	let show = false;

	// Slider stops, left to right. `null` = no parameter sent (model default).
	// 'none' (instant, no reasoning) only when a selected model opts in via `reasoning_effort_none`.
	$: options = [
		{ value: null, label: $i18n.t('fork.reasoningEffort.default') },
		...(supportsNone ? [{ value: 'none', label: $i18n.t('fork.reasoningEffort.none') }] : []),
		{ value: 'low', label: $i18n.t('fork.reasoningEffort.low') },
		{ value: 'medium', label: $i18n.t('fork.reasoningEffort.medium') },
		{ value: 'high', label: $i18n.t('fork.reasoningEffort.high') }
	];

	// Capability defaults to enabled when not explicitly set on the model
	$: enabledModels = (selectedModelIds ?? [])
		.map((id) => ($models ?? []).find((m) => m.id === id))
		.filter((model) => model && model?.info?.meta?.capabilities?.reasoning_effort !== false);
	$: enabled = enabledModels.length > 0;
	$: supportsNone = enabledModels.some(
		(model) => model?.info?.meta?.capabilities?.reasoning_effort_none === true
	);
	$: modelName = enabledModels[0]?.name ?? enabledModels[0]?.id ?? '';

	$: value = params?.reasoning_effort ?? null;
	$: index = Math.max(0, options.findIndex((o) => o.value === value));
	$: selected = options[index];
	$: position = options.length > 1 ? index / (options.length - 1) : 0;
	$: if (disabled && show) show = false;

	const select = (newIndex: number) => {
		const newValue = options[newIndex]?.value ?? null;
		if (newValue === value) return;
		params = { ...(params ?? {}), reasoning_effort: newValue };
	};
</script>

{#if enabled}
	<div class="flex items-center shrink-0">
		<Dropdown bind:show align="end" contentClass="select-none">
			<Tooltip content={$i18n.t('fork.reasoningEffort.label')} placement="top">
				<button
					type="button"
					{disabled}
					aria-disabled={disabled}
					aria-label={$i18n.t('fork.reasoningEffort.label')}
					class="flex items-center gap-1 rounded-lg px-1.5 py-1 text-[0.8125rem] font-normal transition-colors duration-100 {disabled
						? 'cursor-not-allowed opacity-40 text-gray-400 dark:text-gray-600'
						: `text-gray-600 hover:bg-gray-50/40 hover:text-gray-700 dark:text-gray-300 dark:hover:bg-gray-800/40 dark:hover:text-gray-200 ${value ? '' : 'opacity-60'}`}"
				>
					<Bolt className="size-3.5" strokeWidth="2" />
					{#if value}
						<span>{selected.label}</span>
					{/if}
				</button>
			</Tooltip>

			<div
				slot="content"
				class="w-64 rounded-2xl border border-gray-100 bg-white p-3 shadow-lg dark:border-gray-800 dark:bg-gray-850 dark:text-white"
			>
				<div class="relative mb-3 flex items-center justify-center">
					<Bolt className="absolute left-0 size-3.5 text-gray-400" strokeWidth="2" />
					<div class="max-w-[85%] truncate text-sm">
						<span class="font-medium">{modelName}</span>
						<span class="text-gray-400 dark:text-gray-500">{selected.label}</span>
					</div>
				</div>

				<div class="relative h-8 w-full rounded-full bg-gray-100 dark:bg-gray-800">
					<!-- filled part up to the thumb -->
					<div
						class="absolute inset-y-0 left-0 rounded-full bg-blue-500 transition-all duration-150 {value
							? ''
							: 'opacity-0'}"
						style="width: calc({position} * (100% - 2rem) + 2rem);"
					/>

					<!-- stop markers -->
					{#each options as option, i (i)}
						<div
							class="pointer-events-none absolute top-1/2 size-1 -translate-x-1/2 -translate-y-1/2 rounded-full {i <
							index
								? 'bg-white/70'
								: 'bg-gray-400/70 dark:bg-gray-500'}"
							style="left: calc(1rem + {options.length > 1
								? i / (options.length - 1)
								: 0} * (100% - 2rem));"
						/>
					{/each}

					<!-- thumb -->
					<div
						class="pointer-events-none absolute top-0.5 size-7 rounded-full border border-gray-200 bg-white shadow transition-all duration-150 dark:border-gray-600"
						style="left: calc(0.125rem + {position} * (100% - 2rem));"
					/>

					<!-- native range input on top for pointer + keyboard interaction -->
					<input
						type="range"
						min="0"
						max={options.length - 1}
						step="1"
						value={index}
						aria-label={$i18n.t('fork.reasoningEffort.label')}
						aria-valuetext={selected.label}
						class="absolute inset-0 h-full w-full cursor-pointer opacity-0"
						on:input={(e) => select(Number(e.currentTarget.value))}
					/>
				</div>
			</div>
		</Dropdown>
	</div>
{/if}
