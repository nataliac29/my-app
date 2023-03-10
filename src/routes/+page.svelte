<!-- 
USING IBM CARBON SVELTE COMPONENTS: https://carbon-components-svelte.onrender.com/components/Dropdown
CODE FOR DRAGGIN ADAPTED FROM: https://svelte.dev/repl/801ec208ac3a44a5880ea4c31703b39f?version=3.45.0 
CODE FOR MODAL ADAPTED FROM: https://svelte.dev/examples/modal
CODE FOR UPLOADING TIMES TO GOOGLE SHEET ADAPTED FROM: https://github.com/dwyl/learn-to-send-email-via-google-script-html-no-server
-->
<script>
	import {
		Button,
		Modal,
		ModalBody,
		ModalFooter,
		Table,
		Input,
		FormGroup,
		Label,
	} from "sveltestrap";
	import { Dropdown } from "carbon-components-svelte";
	import LocationPreference from "./LocationPreference.svelte";

	let numRows = 30;
	let numDays = 7;
	let columns = new Array(numDays);
	let rows = new Array(numRows);
	let state = new Array(rows.length * columns.length).fill("");
	let freeSeasState = new Array(rows.length * columns.length).fill("free");
	let freeYardState = new Array(rows.length * columns.length).fill("free");
	let name = "";

	const daysOfWeek = ["MON", "TUES", "WED", "THUR", "FRI", "SAT", "SUN"];

	// master list of hours of the day
	const hoursOfDay = [{ id: "0", text: "12:00 AM" }];

	// init hoursOfDay
	for (let i = 1; i <= 23; i++) {
		if (i < 12) {
			// AM times
			hoursOfDay.push({ id: i.toString(), text: `${i}:00 AM` });
		} else if (i == 12) {
			hoursOfDay.push({ id: i.toString(), text: `${i}:00 PM` });
		} else if (i < 24) {
			// PM times
			let j = i % 12;
			hoursOfDay.push({ id: i.toString(), text: `${j}:00 PM` });
		} else {
			hoursOfDay.push({ id: i.toString(), text: `${12}:00 AM` });
		}
	}

	let isDrag = false;
	let isYard = true;
	let isSeas = false;
	let endTime = null;
	let startTime = null;
	let timer = 0;
	let clicked = new Array(rows.length * columns.length).fill(false); // keeps track of selected cells in one click

	// the default times chosen are 9 AM to 12 AM
	let startTimeLabel = 9;
	let endTimeLabel = 23;
	let timeLabels = [];
	let validNewStart = [];
	let validNewEnd = [];

	// concept: user preference on time
	// function for users to choose their start/end times for every day
	const changeTime = () => {
		timeLabels = [];
		for (let i = startTimeLabel; i <= endTimeLabel; i++) {
			if (i < 12 && i < endTimeLabel) {
				// AM times before endTimeLabel
				if (i == 0) {
					timeLabels.push(`12:00 AM`);
				} else {
					timeLabels.push(`${i}:00 AM`);
				}
			} else if (i < endTimeLabel) {
				// PM times before endTimeLabel and not 12 PM
				if (i == 12) {
					timeLabels.push(`${12}:00 PM`);
				} else {
					let j = i % 12;
					timeLabels.push(`${j}:00 PM`);
				}
			} else {
				// endTimeLabel does not add additional x:30
				if (i < 12) {
					timeLabels.push(`${i}:00 AM`);
				} else if (i == 12) {
					timeLabels.push(`${12}:00 PM`);
				}
				else {
					let j = i % 12;
					timeLabels.push(`${j}:00 PM`);
				}
			}
		}
		validNewStart = hoursOfDay.slice(0, endTimeLabel);
		validNewEnd = hoursOfDay.slice(startTimeLabel + 1, 24);
	};

	// initializing times to 9 AM - 12 AM
	changeTime();

	// concept: user preference on time
	// changeStart is a boolean to determine whether the user has changed the start or end time
	// newTime is the variable they want to change it to
	const changeTable = (changeStart, newTime) => {
		let newRows;
		let changeRows;
		let changeState;
		let changeFreeState;
		if (changeStart) {
			if (newTime < startTimeLabel) {
				// change to an earlier start time
				newRows = (startTimeLabel - newTime) * 2;
				numRows += newRows;

				changeRows = new Array(newRows);
				changeState = new Array(newRows * columns.length).fill("");
				changeFreeState = new Array(newRows * columns.length).fill("free");

				rows = changeRows.concat(rows);
				state = changeState.concat(state);
				freeSeasState = changeFreeState.concat(freeSeasState);
				freeYardState = changeFreeState.concat(freeYardState);
			} else {
				// change to later state time
				newRows = (newTime - startTimeLabel) * 2;
				numRows -= newRows;

				rows.splice(0, newRows);
				state.splice(0, newRows);
				freeSeasState.splice(0, newRows);
				freeYardState.splice(0, newRows);
			}
		} else {
			if (newTime < endTimeLabel) {
				// change to earlier end time
				newRows = (endTimeLabel - newTime) * 2;
				numRows -= newRows;

				rows.splice(numRows, newRows);
				state.splice(numRows, newRows);
				freeSeasState.splice(numRows, newRows);
				freeYardState.splice(numRows, newRows);
			} else {
				// change to later end time
				newRows = (newTime - endTimeLabel) * 2;
				numRows += newRows;

				changeRows = new Array(newRows);
				changeState = new Array(newRows * columns.length).fill("");
				changeFreeState = new Array(newRows * columns.length).fill("free");

				rows = changeRows.concat(rows);
				state = changeState.concat(state);
				freeSeasState = changeFreeState.concat(freeSeasState);
				freeYardState = changeFreeState.concat(freeYardState);
			}
		}
	};

	// concept: user preference on time
	const dropdownStart = (e) => {
		if (parseInt(e.detail.selectedId) != startTimeLabel) {
			changeTable(true, parseInt(e.detail.selectedId));
			startTimeLabel = parseInt(e.detail.selectedId);
			changeTime();
		}
	};

	// concept: user preference on time
	const dropdownEnd = (e) => {
		if (parseInt(e.detail.selectedId) != endTimeLabel) {
			changeTable(false, parseInt(e.detail.selectedId));
			endTimeLabel = parseInt(e.detail.selectedId);
			changeTime();
		}
	};

	// concept: user input
	const beginDrag = () => {
		isDrag = true;
		clicked.fill(false);
	};

	// concept: user input
	const endDrag = () => {
		isDrag = false;
		clicked.fill(false);
	};

	// concept: user preference on location
	const changeLocation = () => {
		isYard = !isYard;
		isSeas = !isSeas;
	};

	// concept: rendering of times to meet needs (i.e. providing a buffer for traveling between locations)
	const mouseToggle = (r, c) => {
		let cellIndex = r * columns.length + c;

		let blockBeforeFree = state[(r - 1) * columns.length + c] == "";
		let blockAfterFree = state[(r + 1) * columns.length + c] == "";

		// whether blocks two before and after selected block are free
		let twoBeforeFree = state[(r - 2) * columns.length + c] == "";
		let twoAfterFree = state[(r + 2) * columns.length + c] == "";

		// whether blocks two before and after selected block are yard
		let twoBlocksAfterYard = state[(r + 2) * columns.length + c] == "y";
		let twoBlocksBeforeYard = state[(r - 2) * columns.length + c] == "y";
		
		// whether blocks two before and after selected block are seas
		let twoBlocksAfterSeas = state[(r + 2) * columns.length + c] == "s";
		let twoBlocksBeforeSeas = state[(r - 2) * columns.length + c] == "s";

		// whether blocks before and after selected block are yard
		let blockAfterYard = state[(r + 1) * columns.length + c] == "y";
		let blockBeforeYard = state[(r - 1) * columns.length + c] == "y";
			
		// whether blocks before and after selected block are seas
		let blockAfterSeas = state[(r +1) * columns.length + c] == "s";
		let blockBeforeSeas = state[(r - 1) * columns.length + c] == "s";
				
		if (!clicked[r * columns.length + c]) {
			// if they are marking busy time in SEAS
			if (isSeas && state[r * columns.length + c] == "s") {
				state[r * columns.length + c] = "";

				// if Yard is before or after it, do not free it for SEAS table
				if (!(blockAfterYard || blockBeforeYard)){freeSeasState[r * columns.length + c] = "free"}
				
				// there are 4 cases, if the deleted event is in the middle of an existing block, at the end, at the beginning, or is a singlular block
				// if the block is in the middle, don't change SEAS availability (still unavailable)

				// Case # 1, block is a singular block, or at the ends
				if (
					(blockAfterFree || r + 1 == numRows) &&
					(blockBeforeFree || r == 0)
				) {
					freeYardState[r * columns.length + c] = "free";
					if (twoBeforeFree || r == 1) {
						freeYardState[(r - 1) * columns.length + c] = "free";
					}
					if (twoAfterFree || r == numRows - 2) {
						freeYardState[(r + 1) * columns.length + c] = "free";
					}
				}
				// Case #2, if the block is at the end
				// if at the end, to add a "free" SEAS block below, need to make sure at least an hour to next Yard event
				else if (
					blockAfterFree &&
					(twoAfterFree || twoBlocksAfterYard) &&
					!blockBeforeFree
				) {
					// only free if the "new" end of block is yard
					if (blockBeforeYard){freeYardState[r * columns.length + c] = "free"}
					freeYardState[(r + 1) * columns.length + c] = "free";
					if (twoBeforeFree){freeSeasState[(r + 1) * columns.length + c] = "free"}
					if (r == numRows - 3) {
						freeYardState[(r + 2) * columns.length + c] = "free";
					}
				}
				// Case #3, if the block is in the beginning
				else if (
					// if at the beginning, to add a "free" SEAS block above, need to make sure at least an hour before previous Yard event
					!blockAfterFree &&
					blockBeforeFree &&
					(twoBeforeFree || twoBlocksBeforeYard)
				) {
					// only free for Yard if the "new" beginning of block is yard
					if (blockAfterYard){
						freeYardState[r * columns.length + c] = "free";
					}
					if (twoBeforeFree){freeSeasState[(r - 1) * columns.length + c] = "free"}
					freeYardState[(r - 1) * columns.length + c] = "free";
					if (r == 2) {
						freeYardState[(r - 2) * columns.length + c] = "free";
					}
				}

				// if they are erasing busy yard time
			} else if (isYard && state[r * columns.length + c] == "y") {
				state[r * columns.length + c] = "";
				// if Yard is before or after it, do not free it for SEAS table
				if (!(blockAfterSeas || blockBeforeSeas)){freeYardState[r * columns.length + c] = "free"}
				

				// there are 4 cases, if the deleted event is in the middle of an existing block, at the end, at the beginning, or is a singlular block
				// if the block is in the middle, don't change SEAS availability (still unavailable)

				// Case # 1, block is a singular block, or at the ends
				if (
					(blockAfterFree || r + 1 == numRows) &&
					(blockBeforeFree || r == 0)
				) {
					freeSeasState[r * columns.length + c] = "free";

					if (twoBeforeFree || r == 1) {
						freeSeasState[(r - 1) * columns.length + c] = "free";
					}
					if (twoAfterFree || r == numRows - 2) {
						freeSeasState[(r + 1) * columns.length + c] = "free";
					}
				}
				// Case #2, if the block is at the end
				// if at the end, to add a "free" SEAS block below, need to make sure at least an hour to next Yard event
				else if (
					blockAfterFree &&
					(twoAfterFree || twoBlocksAfterSeas) &&
					!blockBeforeFree
				) {
					// only free for Seas if the "new" end of block is Seas
					if (blockBeforeSeas){freeSeasState[r * columns.length + c] = "free"}
					if (twoBeforeFree){freeYardState[(r + 1) * columns.length + c] = "free"}
					freeSeasState[(r + 1) * columns.length + c] = "free";
					if (r == numRows - 3) {
						freeSeasState[(r + 2) * columns.length + c] = "free";
					}
				}
				// Case #3, if the block is in the beginning
				else if (
					// if at the beginning, to add a "free" SEAS block above, need to make sure at least an hour before previous Yard event
					!blockAfterFree &&
					blockBeforeFree &&
					(twoBeforeFree || twoBlocksBeforeSeas)
				) {
					// only free for Seas if the "new" beginning of block is Seas
					if (blockAfterSeas){freeSeasState[r * columns.length + c] = "free"}
					if (twoBeforeFree){freeYardState[(r - 1) * columns.length + c] = "free"}
					freeSeasState[(r - 1) * columns.length + c] = "free";
					if (r == 2) {
						freeSeasState[(r - 2) * columns.length + c] = "free";
					}
				}
			// if cell is being changed from SEAS -> Yard
			} else if (isYard && state[r * columns.length + c] == "s") {
				state[r * columns.length + c] = "y";
				
				// if the block after is free AND two blocks after is free or is at the Yard,
				// the block after should now be seen as available on Yard availability 
				if (blockAfterFree && (twoAfterFree || twoBlocksAfterYard)){
					freeYardState[(r+1) * columns.length + c] = "free";
					freeSeasState[(r+1) * columns.length + c] = "";
				}

				// if the block before is free AND two before after is free or is at the Yard,
				// the block before should now be seen as available on Yard availability 
				if (blockBeforeFree && (twoBeforeFree || twoBlocksBeforeYard)){
					freeYardState[(r-1) * columns.length + c] = "free";
					freeSeasState[(r-1) * columns.length + c] = "";
				}

			// if cell is being changed from Yard -> SEAS
			}else if (isSeas && state[r * columns.length + c] == "y") {
				state[r * columns.length + c] = "s";
				
				// if the block after is free AND two blocks after is free or is at SEAS,
				// the block after should now be seen as available on SEAS availability 
				if (blockAfterFree && (twoAfterFree || twoBlocksAfterSeas)){
					freeSeasState[(r+1) * columns.length + c] = "free";
					freeYardState[(r+1) * columns.length + c] = "";
				}

				// if the block before is free AND two before after is free or is at the SEAS,
				// the block before should now be seen as available on SEAS availability 
				if (blockBeforeFree && (twoBeforeFree || twoBlocksBeforeSeas)){
					freeSeasState[(r-1) * columns.length + c] = "free";
					freeYardState[(r-1) * columns.length + c] = "";
				}
			}
			// if the cell is marked not busy in the busy calendar, marking busy time
			else {
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

	// concept: user input and location preference
	const mouseHandler = (r, c) => (e) => {
		if (isDrag || e.type === "mousedown") {
			mouseToggle(r, c);
		}
		if (e.type === "mousedown") {
			clicked[r * columns.length + c] = true;
		}
	};

	// concept: user input and location preference
	const seasMouseHandler = (r, c) => (e) => {
		if (isDrag || e.type === "mousedown") {
			seasMouseToggle(r, c);
		}
		if (e.type === "mousedown") {
			clicked[r * columns.length + c] = true;
		}
	};

	// concept: user input
	const seasMouseToggle = (r, c) => {
		freeSeasState[r * columns.length + c] =
			freeSeasState[r * columns.length + c] == "" ? "free" : "";
	};

	// concept: user input
	const yardMouseHandler = (r, c) => (e) => {
		if (isDrag || e.type === "mousedown") {
			yardMouseToggle(r, c);
		}
		if (e.type === "mousedown") {
			clicked[r * columns.length + c] = true;
		}
	};

	// concept: user input
	const yardMouseToggle = (r, c) => {
		freeYardState[r * columns.length + c] =
			freeYardState[r * columns.length + c] == "" ? "free" : "";
	};

	// concept: user input
	// when "DONE" button is clicked
	const clickHandler = () => {
		isOpen = true;
		endTime = Date.now();
		let difference = Math.abs(endTime - startTime);
		timer = difference / 1000;
		var formData = {};
		formData["message"] = timer.toString();
		formData["name"] = name;

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
		state = new Array(rows.length * columns.length).fill("");
		freeSeasState = new Array(rows.length * columns.length).fill("free");
		freeYardState = new Array(rows.length * columns.length).fill("free");
	};

	let isOpen = true;
	let fullscreen = true;
	// concept: user input
	const toggle = () => {
		fullscreen = true;
		isOpen = !isOpen;
		startTime = Date.now();
	};

	// concept: user error
	const clearData = () => {
		if (confirm("Are you sure you want to clear your input across all tables?")){
			state = new Array(rows.length * columns.length).fill("");
			freeSeasState = new Array(rows.length * columns.length).fill("free");
			freeYardState = new Array(rows.length * columns.length).fill("free");
		}
	}
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

<!-- concept: user identification -->
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
		Enter your name, and then click the "Start!" to start the time (or exit the
		modal), and when you're done, click the "Done" button to stop the time
	</ModalBody>
	<ModalFooter>
		<FormGroup>
			<Label for="exampleName">Name</Label>
			<Input
				type="text"
				name="name"
				placeholder="Name"
				required
				bind:value={name}
			/>
		</FormGroup>
		<Button color="primary" on:click={toggle}>Start!</Button>
	</ModalFooter>
</Modal>

<!-- Button for indicating done inputting availability  -->

<h1>Directions:</h1>
<h3>Please input when you are busy on the far left!</h3>

<p class="directions">
	For times when you are in the Yard, click "Yard" and enter the times, and then
	click "SEAS" and enter when you will be at SEAS. We will automatically fill in
	when you are free (as indicated by a green block) in each of the locations.
</p>
<p class="directions">
	We assume that it takes 30 min to get between the Yard (and houses/Quad) and
	SEAS! You can still manually change your available times in each location!
</p>
<p class="directions">
	You can press "q" to toggle more easily! Click "DONE" when you are finished!
</p>
<div class:bigContainer={true}>
	<div>
		<Button
			style="float:left"
			color={isYard ? "primary" : "secondary"}
			disabled={isYard ? true : false}
			on:click={changeLocation}>YARD
		</Button>
		<Button
			style="display:inline-block"
			on:click={changeLocation}
			color={isSeas ? "warning" : "secondary"}
			disabled={isSeas ? true : false}>SEAS
		</Button>
	</div>
	<button style="display:block; justify-content:center; margin: 15px" on:click={clearData}>Reset</button>

	<div class:container={true}>
		<Table bordered style="margin: 10px">
			<caption>Input your busy times into this table.</caption>

			<thead>
				<tr>
					<th class="headers" scope="col" />
					{#each daysOfWeek as day}
						<th class="headers" scope="col">{day}</th>
					{/each}
				</tr>
			</thead>
			<tbody class="main">
				{#each rows as _row, r}
					<tr style="height:12px">
						<!-- svelte concept: control flow -->
						{#if r === 0 && r % 2 != 1}
							<th rowspan="2" class="time-label">
								<!-- concept: user preference on time -->
								<!-- svelte concept: reactive values, event handler, binding -->
								<Dropdown
									size="sm"
									light
									selectedId="9"
									items={validNewStart}
									on:select={dropdownStart}
									bind:value={startTimeLabel}
								/>
							</th>
						{:else if r < rows.length - 2 && r % 2 != 1}
							<th rowspan="2" class="time-label" scope="row"
								>{timeLabels[r / 2]}</th
							>
						{:else if r == rows.length - 2}
							<th rowspan="2" class="time-label">
								<!-- concept: user preference on time -->
								<!-- svelte concept: reactive values, event handler, binding -->
								<Dropdown
									size="sm"
									light
									bind:value={endTimeLabel}
									selectedId="23"
									items={validNewEnd}
									on:select={dropdownEnd}
								/>
							</th>
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
		<!-- concept: user preference on and location -->
		<!-- svelte concept: reusable component, props -->
		<LocationPreference 
			daysOfWeek={daysOfWeek} 
			rows={rows} 
			columns={columns}
			timeLabels={timeLabels} 
			isDrag={isDrag} 
			clicked = {clicked}
			freeLocState={freeSeasState}
			mouseHandler = {seasMouseHandler}
			loc = "SEAS"
		/>
		<!-- concept: user preference on location -->
		<!-- svelte concept: reusable component, props -->
		<LocationPreference 
			daysOfWeek={daysOfWeek} 
			rows={rows} 
			columns={columns}
			timeLabels={timeLabels} 
			isDrag={isDrag} 
			clicked = {clicked}
			freeLocState={freeYardState}
			mouseHandler = {yardMouseHandler}
			loc = "Yard"
		/>
	</div>
	<!-- svelte concept: event handler -->
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
		/* border-color: #979797; */
	}
	tbody {
		border-color: #9797975b;
	}
	.seas {
		background-color: orange;
	}
	.yard {
		background-color: blue;
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
	.directions{
		text-align: center;
	}
</style>
