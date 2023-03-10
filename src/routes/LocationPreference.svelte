<!-- The component for the preference tables for SEAS and the Yard -->
<script>
	import {Table} from "sveltestrap";

	// svelte concept: props
	export let daysOfWeek;
	export let rows;
	export let timeLabels;
	export let freeLocState;
	export let isDrag;
	export let clicked;
	export let columns;
	export let mouseHandler;
	export let loc;

</script>

<Table bordered style="margin: 10px;">
	<!-- svelte concept: reactive values -->
	<caption
		>This reflects when you are free in the {loc}; feel free to adjust to
		your preferences!</caption
	>

	<thead>
		<tr />
		<tr>
			<th class="headers" />
			{#each daysOfWeek as day}
				<th class="headers">{day}</th>
			{/each}
		</tr>
	</thead>
	<tbody>
		{#each rows as _row, r}
			<tr style="height:12px">
				{#if r % 2 != 1}
					<th rowspan="2" class="time-label" scope="row"
						>{timeLabels[r / 2]}</th
					>
				{/if}
				{#each columns as _column, c}
					<td
						on:mousedown={mouseHandler(r, c)}
						on:mouseenter={mouseHandler(r, c)}
						class="cells"
						style="margin: 5px;"
						class:free={freeLocState[r * columns.length + c] == "free"}
					/>
				{/each}
			</tr>
		{/each}
	</tbody>
</Table>


<style>
	td {
		background-color: white;
		padding: 0px;
	}
	tbody {
		border-color: #9797975b;
	}
	.free {
		background-color: green;
	}
	th.time-label {
		position: relative;
		border-style: none;
		top: 0;
		font-weight: bold;
		font-size: xx-small;
		width: 60px;
		height: 15 px;
		user-select: none;
		border: none;
		white-space: nowrap;
	}
	.headers {
		width: 15px;
		font-weight: light;
		border: none;
		user-select: none;
	}
</style>