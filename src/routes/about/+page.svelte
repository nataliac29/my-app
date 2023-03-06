<!-- CODE FOR DRAGGIN ADAPTED FROM: https://svelte.dev/repl/801ec208ac3a44a5880ea4c31703b39f?version=3.45.0 
CODE FOR MODAL ADAPTED FROM: https://svelte.dev/examples/modal
CODE FOR UPLOADING TIMES TO GOOGLE SHEET ADAPTED FROM: https://github.com/dwyl/learn-to-send-email-via-google-script-html-no-server
-->
<script>
	// import Modal from "./Modal.svelte";
	import { Button, Modal, ModalBody, ModalFooter, Table } from "sveltestrap";
	import { Dropdown } from "carbon-components-svelte";

	let showModal = true;
	let numRows = 42;
	let numDays = 7;
	let columns = new Array(numDays);
	let rows = new Array(numRows);
	let state = new Array(rows.length * columns.length).fill("");
	let freeSeasState = new Array(rows.length * columns.length).fill("free");
	let freeYardState = new Array(rows.length * columns.length).fill("free");

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
	let endTime = null;		//???
	let startTime = null;		//???
	let timer = 0;
	let clicked = new Array(rows.length * columns.length).fill(false); // keeps track of selected cells in one click

	let startTimeLabel = 9;
	let endTimeLabel = 24;
	let timeLabels = [];

	const changeTime = () => {
		timeLabels = [];
		for (let i = startTimeLabel; i <= endTimeLabel; i++) {
			if (i < 12 && i < endTimeLabel){
				timeLabels.push(`${i}:00 AM`);
				timeLabels.push(`${i}:30 AM`);
			}
			else if (i < endTimeLabel){
				let j = i%12;
				timeLabels.push(`${j}:00 PM`);
				timeLabels.push(`${j}:30 PM`);
			}
			else if (endTimeLabel != 24){
				let j = i%12;
				timeLabels.push(`${j}:00 PM`);
			}
			else{
				timeLabels.push(`${12}:00 AM`);
			}
		}
	}

	changeTime();
	// startTimeLabel = 13;
	// endTimeLabel = 20;
	// changeTime();

	const people = [
		{ id: 1, name: "Durward Reynolds", unavailable: false },
		{ id: 2, name: "Kenton Towne", unavailable: false },
		{ id: 3, name: "Therese Wunsch", unavailable: false },
		{ id: 4, name: "Benedict Kessler", unavailable: true },
		{ id: 5, name: "Katelyn Rohan", unavailable: false },
	];

	let selectedPerson = people[0];

	const beginDrag = () => {
		isDrag = true;
		clicked.fill(false);
	};

	const endDrag = () => {
		isDrag = false;
		clicked.fill(false);
	};

	const changeLocation = () => {
		isYard = !isYard;
		isSeas = !isSeas;
	};

	const mouseToggle = (r, c) => {
		let cellIndex = r * columns.length + c;

		let blockBeforeNotBusy = state[(r - 1) * columns.length + c] == "";
		let blockAfterNotBusy = state[(r + 1) * columns.length + c] == "";
		let twoBlocksBeforeNotBusy = state[(r - 2) * columns.length + c] == "";
		let twoBlocksAfterNotBusy = state[(r + 2) * columns.length + c] == "";
		if (!clicked[r * columns.length + c]) {
			// if they are marking busy time in SEAS
			if (isSeas && state[r * columns.length + c] == "s") {
				state[r * columns.length + c] = "";

				// there are 4 cases, if the deleted event is in the middle of an existing block, at the end, at the beginning, or is a singlular block
				// if the clock is in the middle, don't change SEAS availability (still unavailable)

				console.log("THIS IS R");
				console.log(r);
				// Case # 1, block is a singular block, or at the ends
				if (
					(blockAfterNotBusy || r + 1 == numRows) &&
					(blockBeforeNotBusy || r == 0)
				) {
					freeYardState[r * columns.length + c] = "free";
					console.log("HERE IN CASE 1");

					console.log("twoBlocksBeforeNotBusy");
					console.log(twoBlocksBeforeNotBusy);
					console.log("twoBlocksAfterNotBusy");
					console.log(twoBlocksAfterNotBusy);
					if (twoBlocksBeforeNotBusy || r == 1) {
						freeYardState[(r - 1) * columns.length + c] = "free";
					}
					if (twoBlocksAfterNotBusy || r == numRows - 2) {
						freeYardState[(r + 1) * columns.length + c] = "free";
					}
				}
				// Case #2, if the block is at the end
				// if at the end, to add a "free" SEAS block below, need to make sure at least an hour to next Yard event
				else if (
					blockAfterNotBusy &&
					twoBlocksAfterNotBusy &&
					!blockBeforeNotBusy
				) {
					console.log("HERE IN CASE 2");
					freeYardState[(r + 1) * columns.length + c] = "free";
					if (r == numRows - 3) {
						freeYardState[(r + 2) * columns.length + c] = "free";
					}
				}
				// Case #3, if the block is in the beginning
				else if (
					// if at the beginning, to add a "free" SEAS block above, need to make sure at least an hour before previous Yard event
					!blockAfterNotBusy &&
					blockBeforeNotBusy &&
					twoBlocksBeforeNotBusy
				) {
					console.log("HERE IN CASE 3");

					freeYardState[(r - 1) * columns.length + c] = "free";
					if (r == 2) {
						freeYardState[(r - 2) * columns.length + c] = "free";
					}
				} else {
					console.log("HERE AFTER CASE 3");
				}

				// if they are erasing busy yard time
			} else if (isYard && state[r * columns.length + c] == "y") {
				state[r * columns.length + c] = "";

				// there are 4 cases, if the deleted event is in the middle of an existing block, at the end, at the beginning, or is a singlular block
				// if the clock is in the middle, don't change SEAS availability (still unavailable)

				console.log("THIS IS R");
				console.log(r);
				// Case # 1, block is a singular block, or at the ends
				if (
					(blockAfterNotBusy || r + 1 == numRows) &&
					(blockBeforeNotBusy || r == 0)
				) {
					freeSeasState[r * columns.length + c] = "free";
					console.log("HERE IN CASE 1");

					console.log("twoBlocksBeforeNotBusy");
					console.log(twoBlocksBeforeNotBusy);
					console.log("twoBlocksAfterNotBusy");
					console.log(twoBlocksAfterNotBusy);
					if (twoBlocksBeforeNotBusy || r == 1) {
						freeSeasState[(r - 1) * columns.length + c] = "free";
					}
					if (twoBlocksAfterNotBusy || r == numRows - 2) {
						freeSeasState[(r + 1) * columns.length + c] = "free";
					}
				}
				// Case #2, if the block is at the end
				// if at the end, to add a "free" SEAS block below, need to make sure at least an hour to next Yard event
				else if (
					blockAfterNotBusy &&
					twoBlocksAfterNotBusy &&
					!blockBeforeNotBusy
				) {
					console.log("HERE IN CASE 2");
					freeSeasState[(r + 1) * columns.length + c] = "free";
					if (r == numRows - 3) {
						freeSeasState[(r + 2) * columns.length + c] = "free";
					}
				}
				// Case #3, if the block is in the beginning
				else if (
					// if at the beginning, to add a "free" SEAS block above, need to make sure at least an hour before previous Yard event
					!blockAfterNotBusy &&
					blockBeforeNotBusy &&
					twoBlocksBeforeNotBusy
				) {
					console.log("HERE IN CASE 3");

					freeSeasState[(r - 1) * columns.length + c] = "free";
					if (r == 2) {
						freeSeasState[(r - 2) * columns.length + c] = "free";
					}
				} else {
					console.log("HERE AFTER CASE 3");
				}
				// if the cell is marked not busy in the busy calendar, marking busy time
			} else {
				state[r * columns.length + c] = isSeas ? "s" : "y";
				if (isYard) {
					freeSeasState[(r + 1) * columns.length + c] = "";
					freeSeasState[(r - 1) * columns.length + c] = "";
					freeSeasState[r * columns.length + c] = "";

					freeYardState[r * columns.length + c] = "";
				}
				if (isSeas) {
					freeYardState[(r + 1) * columns.length + c] = "";
					freeYardState[(r - 1) * columns.length + c] = "";
					freeYardState[r * columns.length + c] = "";

					freeSeasState[r * columns.length + c] = "";
				}

				clicked[r * columns.length + c] = true;
			}
		}
	};

	const mouseHandler = (r, c) => (e) => {
		if (isDrag || e.type === "mousedown") {
			mouseToggle(r, c);
		}
		if (e.type === "mousedown") {
			clicked[r * columns.length + c] = true;
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
		<Table bordered style="margin: 10px">
			<caption>When are you busy?</caption>

			<thead>
				<tr>
					<th scope="col"></th>
					{#each daysOfWeek as day}
						<th scope="col">{day}</th>
					{/each}
				</tr>
			</thead>
			<tbody>
				{#each rows as _row, r}
					<tr>
						{#if r === 0}
							<th>
								<Dropdown
								light
								selectedId = "9"
								items={[
									{ id: "9", text: "9:00 AM" },
									{ id: "10", text: "10:00 AM" },
									{ id: "11", text: "11:00 AM" },
								]}
								/>
							</th>
						{:else if r < rows.length}
							<th scope="row">{timeLabels[r]}</th>
						{:else}
							<th scope="row">hi</th>
						{/if}
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
		<Table bordered style="margin: 10px">
			<caption
				>When you are free in SEAS, feel free to adjust to your preferences!</caption
			>

			<thead>
				<tr>
					{#each daysOfWeek as day, i}
						<td>{day}</td>
					{/each}
				</tr>
			</thead>
			<tbody>
				{#each rows as _row, r}
					<tr style="height:12px">
						{#each columns as _column, c}
							<td
								style="margin: 5px;"
								class:free={freeSeasState[r * columns.length + c] == "free"}
							/>
						{/each}
					</tr>
				{/each}
			</tbody>
		</Table>
		<Table bordered style="margin: 10px;">
			<caption
				>When you are free in the Yard, feel free to adjust to your preferences!</caption
			>

			<thead>
				<tr>
					{#each daysOfWeek as day, i}
						<td>{day}</td>
					{/each}
				</tr>
			</thead>
			<tbody>
				{#each rows as _row, r}
					<tr style="height:12px">
						{#each columns as _column, c}
							<td
								style="margin: 5px;"
								class:free={freeYardState[r * columns.length + c] == "free"}
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
	.availableContainer {
		display: flex;
		flex-direction: column;
	}
</style>
