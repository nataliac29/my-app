<!-- CODE FOR DRAGGIN ADAPTED FROM: https://svelte.dev/repl/801ec208ac3a44a5880ea4c31703b39f?version=3.45.0 
CODE FOR MODAL ADAPTED FROM: https://svelte.dev/examples/modal
CODE FOR UPLOADING TIMES TO GOOGLE SHEET ADAPTED FROM: https://github.com/dwyl/learn-to-send-email-via-google-script-html-no-server
-->
<script>
	// import Modal from "./Modal.svelte";
	import { Button, Modal, ModalBody, ModalFooter, Table } from "sveltestrap";
	let showModal = true;

	let columns = new Array(7);
	let rows = new Array(42);
	let state = new Array(rows.length * columns.length).fill("");
	let freeState = new Array(rows.length * columns.length).fill("free");

	let daysOfWeek = [
		"Monday",
		"Tuesday",
		"Wednesday",
		"Thursday",
		"Friday",
		"Saturday",
		"Sunday",
	];
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

	const mouseToggle = (r, c) => {
		if (isSeas && state[r * columns.length + c] == "s") {
			state[r * columns.length + c] = "";
			freeState[r * columns.length + c] = "free";
		} else if (isYard && state[r * columns.length + c] == "y") {
			state[r * columns.length + c] = "";
			freeState[r * columns.length + c] = "free";
		} else {
			state[r * columns.length + c] = isSeas ? "s" : "y";
			freeState[r * columns.length + c] = "";
		}
	};

	const mouseHandler = (r, c) => (e) => {
		if (isDrag || e.type === "mousedown") {
			mouseToggle(r, c);
		}
	};

	// when "DONE" button is clicked
	const clickHandler = () => {
		isOpen = true;
		endTime = Date.now();
		let difference = Math.abs(endTime - startTime);
		timer = difference / 1000;
		var formData = {};
		formData["message"] = timer.toString();
		formData.formDataNameOrder = JSON.stringify(["message"]);
		formData.formGoogleSheetName = "responses"; // default sheet name
		formData.formGoogleSendEmail = "ncalvo@college.harvard.edu"; // no email by default

		// link to Google script that uploads data to Google Sheet
		var url =
			"https://script.google.com/macros/s/AKfycbxmP2lKcfwf10uL-Ghknowoe2S-E9IHtFY5ZhQrG1QCNQdK-ucPqE4QUud_oHDFrE6pdA/exec";
		var xhr = new XMLHttpRequest();

		xhr.open("POST", url);
		xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
		// url encode form data for sending as post data
		var encoded = Object.keys(formData)
			.map(function (k) {
				return encodeURIComponent(k) + "=" + encodeURIComponent(formData[k]);
			})
			.join("&");
		xhr.send(encoded);
	};

	let isOpen = true;
	let fullscreen = true;
	const toggle = () => {
		fullscreen = true;
		isOpen = !isOpen;
		startTime = Date.now();
	};
</script>

<svelte:head>
	<title>About</title>
	<meta name="description" content="About this app" />
	<link
		rel="stylesheet"
		href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.1/dist/css/bootstrap.min.css"
	/>
</svelte:head>

<svelte:window
	on:mousedown={beginDrag}
	on:mouseup={endDrag}
	on:keypress={(e) => e.key == "q" && changeLocation()}
/>
<div hidden={!isOpen}>
	<Button color="primary" on:click={toggle}>Hello World!</Button>
</div>
<Modal
	body
	{isOpen}
	{toggle}
	header="Start your timer!"
	backdrop="static"
	keyboard={false}
	{fullscreen}
	class="modal-dialog"
>
	<ModalBody>
		Click the "Start!" to start the time (or exit the modal), and when you're
		done, click the "Done" button to stop the time
	</ModalBody>
	<ModalFooter>
		<Button color="primary" on:click={toggle}>Start!</Button>
	</ModalFooter>
</Modal>

<!-- Button for indicating done inputting availability  -->
<div class:bigContainer={true}>
	<Button
		color={isYard ? "primary" : "secondary"}
		disabled={isYard ? true : false}
		on:click={changeLocation}>YARD</Button
	>
	<Button
		on:click={changeLocation}
		color={isSeas ? "success" : "secondary"}
		disabled={isSeas ? true : false}>SEAS</Button
	>

	<div class:container={true}>
		<Table bordered>
			<caption>When are you busy?</caption>

			<thead>
				<tr>
					{#each daysOfWeek as day, i}
						<td>{day}</td>
					{/each}
				</tr>
			</thead>
			<tbody>
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
			</tbody>
		</Table>
		<Table bordered>
			<caption>When you are free!</caption>

			<thead>
				<tr>
					{#each daysOfWeek as day, i}
						<td>{day}</td>
					{/each}
				</tr>
			</thead>
			<tbody>
				{#each rows as _row, r}
					<tr style="height:13px">
						{#each columns as _column, c}
							<td
								style="margin: 5px;"
								class:free={freeState[r * columns.length + c] == "free"}
							/>
						{/each}
					</tr>
				{/each}
			</tbody>
		</Table>
	</div>
	<Button on:click={clickHandler} type="submit" color="danger" size="lg"
		>DONE</Button
	>
</div>

<style>
	.bigContainer {
		width: 100%;
		max-width: 1200px;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	.container {
		display: flex;
		flex-direction: row;
		justify-content: center;
		width: 1200px;
	}
	td {
		background-color: white;
		padding: 0px;
	}
	.seas {
		background-color: green;
	}
	.yard {
		background-color: blue;
	}
	.free {
		background-color: yellow;
	}
</style>
