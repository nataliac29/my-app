<!-- CODE FOR DRAGGIN ADAPTED FROM: https://svelte.dev/repl/801ec208ac3a44a5880ea4c31703b39f?version=3.45.0 
CODE FOR MODAL ADAPTED FROM: https://svelte.dev/examples/modal
CODE FOR UPLOADING TIMES TO GOOGLE SHEET FROM: https://github.com/dwyl/learn-to-send-email-via-google-script-html-no-server
-->
<script>
	// import Modal from "./Modal.svelte";
	import { Button, Modal, ModalBody, ModalFooter, Table } from "sveltestrap";
	let showModal = true;

	let columns = new Array(7);
	let rows = new Array(30);
	let state = new Array(rows.length * columns.length).fill("");
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
		} else if (isYard && state[r * columns.length + c] == "y") {
			state[r * columns.length + c] = "";
		} else {
			state[r * columns.length + c] = isSeas ? "s" : "y";
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

<Button on:click={clickHandler} type="submit" color="danger" size="sm"
	>DONE</Button
>

<div class:container={true}>
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

	<Table bordered>
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
							style="margin: 5px;"
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
</div>

<style>
	td {
		width: 20px;
		height: 20px;
		background-color: pink;
		margin-right: 5px;
	}
	.seas {
		background-color: green;
	}
	.yard {
		background-color: blue;
	}
</style>
