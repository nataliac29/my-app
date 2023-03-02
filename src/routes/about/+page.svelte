<script>
	let columns = new Array(7);
	let rows = new Array(30);
	let state = new Array(rows.length * columns.length).fill("");
	let isDrag = false;
	let isYard = true;
	let isSeas = false;

	const beginDrag = () => {
		isDrag = true;
	};

	const endDrag = () => {
		isDrag = false;
	};

	const changeLocation = () => {
		isYard = !isYard;
		isSeas = !isSeas;
	};

	const toggle = (r, c) => {
		if (isSeas && state[r * columns.length + c] == "s") {
			state[r * columns.length + c] = "";
		} else if (isYard && state[r * columns.length + c] == "y") {
			state[r * columns.length + c] = "";
		} else {
			state[r * columns.length + c] = isSeas ? "s" : "y";
		}
	};

	const mouseHandler = (r, c) => (e) => {
		if (isDrag || e.type === "mousedown") {
			toggle(r, c);
		}
	};
</script>

<svelte:head>
	<title>About</title>
	<meta name="description" content="About this app" />
</svelte:head>

<svelte:window
	on:mousedown={beginDrag}
	on:mouseup={endDrag}
	on:keypress={(e) => e.key == "q" && changeLocation()}
/>
<div class:container={true}>
	<button on:click={changeLocation} class:button-selected={isYard}>
		YARD
	</button>
	<button on:click={changeLocation} class:button-selected={isSeas}>
		SEAS
	</button>

	<table>
		{#each rows as _row, r}
			<tr>
				{#each columns as _column, c}
					<td
						on:mousedown={mouseHandler(r, c)}
						on:mouseenter={mouseHandler(r, c)}
						class:seas={state[r * columns.length + c] == "s"}
						class:yard={state[r * columns.length + c] == "y"}
					/>
				{/each}
			</tr>
		{/each}
	</table>
</div>

<style>
	.container {
		width: 100%;
		display: flex;
		flex-direction: row;
	}
	td {
		width: 20px;
		height: 20px;
		background-color: pink;
	}
	.seas {
		background-color: green;
	}
	.yard {
		background-color: blue;
	}
	table {
		border-spacing: 1px;
		width: 100%;
	}
	.button-selected {
		background-color: green;
	}
	button {
		background-color: red;
	}
</style>
