<script lang="ts">
	import { buildRiskMatrix } from './utils';
	import Cell from './Cell.svelte';

	import * as m from '../../../paraglide/messages';
	import type { ComponentType } from 'svelte';
	import { popup, type PopupSettings } from '@skeletonlabs/skeleton';
	import { isDark } from '$lib/utils/helpers';

	export let riskMatrix;
	export let wrapperClass: string | undefined = '';
	export let matrixName; // used to differentiate bubbles tooltip names

	const parsedRiskMatrix = JSON.parse(riskMatrix.json_definition);
	const grid = parsedRiskMatrix.grid;
	const risk = parsedRiskMatrix.risk;
	export let showRisks = false;
	export let useBubbles = false;

	const displayedRiskMatrix = buildRiskMatrix(grid, risk);
	export let data: Array<any> | undefined = undefined;
	export let dataItemComponent: ComponentType | undefined = undefined;
	// reverse data array to display it in the right order
	let displayedData: typeof data;
	if (data) {
		displayedData = data.some((e) => e.length > 0) ? data.slice().reverse() : undefined;
	}
	let popupHover: PopupSettings[][] = [];
	popupHover[0] = [];
	for (let i = 0; i < parsedRiskMatrix.impact.length; i++) {
		popupHover[0].push({
			event: 'hover',
			target: 'popup' + 'impact' + i,
			placement: 'bottom'
		});
	}
	popupHover[1] = [];
	for (let i = 0; i < parsedRiskMatrix.probability.length; i++) {
		popupHover[1].push({
			event: 'hover',
			target: 'popup' + 'probability' + i,
			placement: 'bottom'
		});
	}

	$: classesCellText = (backgroundHexColor: string) => {
		return isDark(backgroundHexColor) ? 'text-white' : '';
	};
</script>

<div class="flex flex-row items-center">
	<div class="flex font-semibold text-xl -rotate-90">{m.probability()}</div>
	<div
		class="{wrapperClass} grid gap-1 w-full"
		style="grid-template-columns: repeat({displayedRiskMatrix[0].length + 1}, minmax(0, 1fr));"
		data-testid="risk-matrix"
	>
		{#each displayedRiskMatrix as row, i}
			{@const reverseIndex = displayedRiskMatrix.length - i - 1}
			{@const probability = parsedRiskMatrix.probability[reverseIndex]}
			<div
				class="flex flex-col items-center h-20 justify-center bg-gray-200 border-dotted border-black border-2 text-center {classesCellText(
					probability.hexcolor ?? '#FFFFFF'
				)}"
				style="background: {probability.hexcolor}"
				data-testid="probability-row-header"
			>
				<div
					class="card bg-black text-gray-200 p-4 z-20"
					style="color: {probability.hexcolor}"
					data-popup={'popup' + 'probability' + i}
				>
					<p data-testid="probability-description" class="font-semibold">
						{probability.description}
					</p>
					<div class="arrow bg-black" />
				</div>
				<span class="font-semibold p-1" data-testid="probability-name">{probability.name}</span>
				{#if probability.description}
					<i class="fa-solid fa-circle-info [&>*]:pointer-events-none" use:popup={popupHover[1][i]}
					></i>
				{/if}
			</div>
			{#each row as cell, j}
				{#if displayedData}
					<Cell
						{cell}
						cellData={displayedData[i][j]}
						popupTarget={`popupdata-${matrixName}-${i}-${j}`}
						{dataItemComponent}
						{useBubbles}
					/>
				{:else}
					<Cell {cell} {dataItemComponent} />
				{/if}
			{/each}
		{/each}
		<div />
		{#each parsedRiskMatrix.impact as impact, key}
			<div
				class="flex flex-col items-center justify-center bg-gray-200 h-20 border-dotted border-black border-2 text-center {classesCellText(
					impact.hexcolor ?? '#FFFFFF'
				)}"
				style="background: {impact.hexcolor}"
				data-testid="impact-col-header"
			>
				<div
					class="card bg-black text-gray-200 p-4 z-20"
					style="color: {impact.hexcolor}"
					data-popup={'popup' + 'impact' + key}
				>
					<p data-testid="impact-description" class="font-semibold">{impact.description}</p>
					<div class="arrow bg-black" />
				</div>
				<span class="font-semibold p-1" data-testid="impact-name">{impact.name}</span>
				{#if impact.description}
					<i
						class="fa-solid fa-circle-info [&>*]:pointer-events-none"
						use:popup={popupHover[0][key]}
					></i>
				{/if}
			</div>
		{/each}
	</div>
</div>
<div class="flex font-semibold text-xl items-center justify-center p-2 pl-60">{m.impact()}</div>
{#if showRisks}
	<div class="w-full flex flex-col justify-start">
		<h3 class="flex font-semibold p-2 m-2 text-md">{m.riskLevels()}</h3>
		<div class="flex justify-start mx-2">
			<table class="w-3/4 border-separate">
				{#each parsedRiskMatrix.risk as risk}
					<tr class="col">
						<td
							class="w-16 text-center border-4 border-white p-2 font-semibold whitespace-nowrap {classesCellText(
								risk.hexcolor
							)}"
							style="background-color: {risk.hexcolor}"
						>
							{risk.name}
						</td>
						<td class="col italic">
							{risk.description}
						</td>
					</tr>
				{/each}
			</table>
		</div>
	</div>
{/if}
