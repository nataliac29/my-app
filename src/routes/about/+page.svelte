<!-- CODE ADAPTED FROM: https://svelte.dev/repl/801ec208ac3a44a5880ea4c31703b39f?version=3.45.0 -->
<script>
	import Modal from "./Modal.svelte";

	let showModal = true;

	let columns = new Array(7);
	let rows = new Array(30);
	let state = new Array(rows.length * columns.length).fill("");
	let isDrag = false;
	let isYard = true;
	let isSeas = false;
	let endTime = null;
	let startTime = null;
	let timer = 0;

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

	const clickHandler = () => {
		showModal = true;
		endTime = Date.now();
		// console.log("HERE, DONE CLICK HANDLER");
		// console.log("START TIME");
		// console.log(startTime);
		// console.log("END TIME");
		// console.log(endTime);
		let difference = Math.abs(endTime - startTime);
		timer = difference / 1000;
		// console.log("DIFFERENCE IN SECONDS");
		// console.log(timer);
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
<form
	class="gform"
	method="POST"
	data-email="ncalvo@college.harvard.edu"
	action="https://script.google.com/macros/s/AKfycbxmP2lKcfwf10uL-Ghknowoe2S-E9IHtFY5ZhQrG1QCNQdK-ucPqE4QUud_oHDFrE6pdA/exec"
>
	<fieldset class="pure-group">
		<textarea id="message" name="message" rows="0" value={timer} />
		>
	</fieldset>
	<button on:click={clickHandler} type="submit"> DONE </button>
	<script
		data-cfasync="false"
		type="text/javascript"
		src="form-submission-handler.js"
	></script>
</form>

<Modal bind:showModal bind:startTime>
	<p>
		Click the "Start Button" to start the time, and when you're done, click the
		"done" button to stop the time
	</p>
</Modal>

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
