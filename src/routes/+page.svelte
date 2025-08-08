<script>
	import FCFSComponent from "$lib/components/FCFSComponent.svelte";
	import RoundRobinComponent from "$lib/components/RoundRobinComponent.svelte";
	import SJFComponent from "$lib/components/SJFComponent.svelte";
	import SRTComponent from "$lib/components/SRTComponent.svelte";

	let selectedAlgorithm = "FCFS";
	let pid = "";
	let burstTime = "";
	let arrivalTime = "";
	let processes = [];
	let showResults = false;

	const algorithms = [
		{ value: "FCFS", label: "First Come First Serve (FCFS)" },
		{ value: "SJF", label: "Shortest Job First (SJF)" },
		{ value: "SRT", label: "Shortest Remaining Time (SRT)" },
		{ value: "RR", label: "Round Robin" },
	];

	// Hide results when algorithm changes
	$: if (selectedAlgorithm) {
		showResults = false;
	}

	function addProcess() {
		// Auto-assign PID if not provided
		if (!pid || pid === "") {
			// Find the smallest available PID starting from 1
			let autoPid = 1;
			while (processes.some((p) => p.pid === autoPid)) {
				autoPid++;
			}
			pid = autoPid.toString();
		}

		// Default arrival time to 0 if not provided
		if (!arrivalTime || arrivalTime === "") {
			arrivalTime = "0";
		}

		if (pid && burstTime && arrivalTime !== "") {
			const pidNum = parseInt(pid);
			const burstNum = parseInt(burstTime);
			const arrivalNum = parseInt(arrivalTime);

			// Check for duplicate PID
			if (processes.some((p) => p.pid === pidNum)) {
				alert(`Process with PID ${pidNum} already exists!`);
				return;
			}

			// Validate values
			if (pidNum <= 0 || burstNum < 0 || arrivalNum < 0) {
				alert("Please enter valid values (PID > 0, Burst Time >= 0, Arrival Time >= 0)");
				return;
			}

			processes = [
				...processes,
				{
					pid: pidNum,
					burstTime: burstNum,
					arrivalTime: arrivalNum,
				},
			];

			// Clear inputs
			pid = "";
			burstTime = "";
			arrivalTime = "";
			showResults = false; // Hide results when new process is added
		} else {
			alert("Please fill burst time field");
		}
	}

	function handleKeyPress(event) {
		if (event.key === "Enter") {
			addProcess();
		}
	}

	function clearProcesses() {
		processes = [];
		showResults = false;
	}

	function addExampleData() {
		processes = [
			{ pid: 1, burstTime: Math.floor(Math.random() * 10) + 1, arrivalTime: Math.floor(Math.random() * 11) },
			{ pid: 2, burstTime: Math.floor(Math.random() * 10) + 1, arrivalTime: Math.floor(Math.random() * 11) },
			{ pid: 3, burstTime: Math.floor(Math.random() * 10) + 1, arrivalTime: Math.floor(Math.random() * 11) },
			{ pid: 4, burstTime: Math.floor(Math.random() * 10) + 1, arrivalTime: Math.floor(Math.random() * 11) },
		];
		showResults = false;
	}

	function removeProcess(index) {
		processes = processes.filter((_, i) => i !== index);
		showResults = false;
	}

	function updateProcess(index, field, value) {
		const newValue = parseInt(value) || 0;

		// Validate PID uniqueness when updating PID
		if (field === "pid") {
			if (processes.some((p, i) => i !== index && p.pid === newValue)) {
				alert(`Process with PID ${newValue} already exists!`);
				return;
			}
			if (newValue <= 0) {
				alert("PID must be greater than 0");
				return;
			}
		}

		// Validate burst time
		if (field === "burstTime" && newValue < 0) {
			alert("Burst time cannot be negative");
			return;
		}

		// Validate arrival time
		if (field === "arrivalTime" && newValue < 0) {
			alert("Arrival time cannot be negative");
			return;
		}

		processes[index][field] = newValue;
		processes = [...processes]; // Trigger reactivity
		showResults = false; // Hide results when processes change
	}

	function calculateResults() {
		if (processes.length === 0) {
			alert("Please add some processes first");
			return;
		}
		showResults = true;
	}
</script>

<div class="container mx-auto p-6 max-w-6xl">
	<h1 class="text-3xl font-bold text-center mb-8 text-white">CPU Scheduling Algorithms</h1>

	<!-- Algorithm Selection -->
	<div class="rounded-lg shadow-lg p-6 mb-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
		<h2 class="text-xl font-semibold mb-4 text-white">Select Scheduling Algorithm</h2>
		<select
			bind:value={selectedAlgorithm}
			class="w-full p-3 rounded-md focus:outline-none focus:ring-2 text-white"
			style="background-color: #434c5e; border: 1px solid #4c566a; focus:ring-color: #88c0d0;"
		>
			{#each algorithms as algorithm}
				<option value={algorithm.value} style="background-color: #434c5e; color: #d8dee9;">{algorithm.label}</option>
			{/each}
		</select>
	</div>

	<!-- Process Input Form -->
	<div class="rounded-lg shadow-lg p-6 mb-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
		<h2 class="text-xl font-semibold mb-4 text-white">Add Process</h2>
		<div class="grid grid-cols-1 md:grid-cols-4 gap-4">
			<div>
				<label for="pid" class="block text-sm font-medium mb-2" style="color: #e5e9f0;">Process ID</label>
				<input
					id="pid"
					type="number"
					bind:value={pid}
					placeholder="Auto-assign if empty"
					min="1"
					on:keypress={handleKeyPress}
					class="w-full p-3 rounded-md focus:outline-none focus:ring-2 text-white placeholder-gray-400"
					style="background-color: #434c5e; border: 1px solid #4c566a; focus:ring-color: #88c0d0;"
				/>
			</div>
			<div>
				<label for="burst" class="block text-sm font-medium mb-2" style="color: #e5e9f0;"
					>Burst Time <span style="color: #bf616a;">*</span></label
				>
				<input
					id="burst"
					type="number"
					bind:value={burstTime}
					placeholder="Enter burst time"
					min="1"
					on:keypress={handleKeyPress}
					class="w-full p-3 rounded-md focus:outline-none focus:ring-2 text-white placeholder-gray-400"
					style="background-color: #434c5e; border: 1px solid #4c566a; focus:ring-color: #88c0d0;"
				/>
			</div>
			<div>
				<label for="arrival" class="block text-sm font-medium mb-2" style="color: #e5e9f0;">Arrival Time</label>
				<input
					id="arrival"
					type="number"
					bind:value={arrivalTime}
					placeholder="Default: 0"
					min="0"
					on:keypress={handleKeyPress}
					class="w-full p-3 rounded-md focus:outline-none focus:ring-2 text-white placeholder-gray-400"
					style="background-color: #434c5e; border: 1px solid #4c566a; focus:ring-color: #88c0d0;"
				/>
			</div>
			<div class="flex items-end gap-2">
				<button
					on:click={addProcess}
					class="flex-1 text-white p-3 rounded-md hover:opacity-90 focus:outline-none focus:ring-2 transition-all duration-200"
					style="background-color: #88c0d0; focus:ring-color: #5e81ac;"
				>
					Add Process
				</button>
				<button
					on:click={addExampleData}
					class="text-white px-4 py-3 rounded-md hover:opacity-90 focus:outline-none focus:ring-2 transition-all duration-200 text-sm"
					style="background-color: #4c566a; focus:ring-color: #434c5e;"
					title="Add example processes"
				>
					Example
				</button>
			</div>
		</div>
		<p class="text-sm mt-3" style="color: #d8dee9;">
			💡 <strong>Tips:</strong> Press Enter to add process • PID auto-assigns if empty • Arrival time defaults to 0 • Only Burst Time is required
		</p>
	</div>

	<!-- Process List -->
	{#if processes.length > 0}
		<div class="rounded-lg shadow-lg p-6 mb-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
			<div class="flex justify-between items-center mb-4">
				<h2 class="text-xl font-semibold text-white">Current Processes ({processes.length})</h2>
				<div class="flex gap-2">
					<button
						on:click={clearProcesses}
						class="text-white px-4 py-2 rounded-md hover:opacity-90 transition-all duration-200"
						style="background-color: #bf616a;"
					>
						Clear All
					</button>
				</div>
			</div>
			<div class="overflow-x-auto">
				<table class="w-full border-collapse rounded-lg overflow-hidden" style="border: 1px solid #4c566a;">
					<thead style="background-color: #434c5e;">
						<tr>
							<th class="px-4 py-2 text-white font-semibold" style="border: 1px solid #4c566a;">PID</th>
							<th class="px-4 py-2 text-white font-semibold" style="border: 1px solid #4c566a;">Burst Time</th>
							<th class="px-4 py-2 text-white font-semibold" style="border: 1px solid #4c566a;">Arrival Time</th>
							<th class="px-4 py-2 text-white font-semibold" style="border: 1px solid #4c566a;">Action</th>
						</tr>
					</thead>
					<tbody>
						{#each processes as process, index}
							<tr class="hover:opacity-90 transition-all duration-200" style="background-color: #2e3440;">
								<td class="px-2 py-2 text-center" style="border: 1px solid #4c566a;">
									<input
										type="number"
										value={process.pid}
										on:input={(e) => updateProcess(index, "pid", e.target.value)}
										class="w-full text-center border-0 bg-transparent focus:outline-none rounded px-2 py-1 text-white"
										style="focus:background-color: #434c5e;"
										min="1"
									/>
								</td>
								<td class="px-2 py-2 text-center" style="border: 1px solid #4c566a;">
									<input
										type="number"
										value={process.burstTime}
										on:input={(e) => updateProcess(index, "burstTime", e.target.value)}
										class="w-full text-center border-0 bg-transparent focus:outline-none rounded px-2 py-1 text-white"
										style="focus:background-color: #434c5e;"
										min="0"
									/>
								</td>
								<td class="px-2 py-2 text-center" style="border: 1px solid #4c566a;">
									<input
										type="number"
										value={process.arrivalTime}
										on:input={(e) => updateProcess(index, "arrivalTime", e.target.value)}
										class="w-full text-center border-0 bg-transparent focus:outline-none rounded px-2 py-1 text-white"
										style="focus:background-color: #434c5e;"
										min="0"
									/>
								</td>
								<td class="px-4 py-2 text-center" style="border: 1px solid #4c566a;">
									<button
										on:click={() => removeProcess(index)}
										class="text-white px-3 py-1 rounded text-sm hover:opacity-90 transition-all duration-200"
										style="background-color: #bf616a;"
									>
										Remove
									</button>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
			<p class="text-sm mt-2" style="color: #d8dee9;">💡 Tip: Click on any value in the table to edit it directly</p>

			<!-- Calculate Button -->
			<div class="mt-4 text-center">
				<button
					on:click={calculateResults}
					class="text-white px-6 py-3 rounded-md hover:opacity-90 focus:outline-none focus:ring-2 transition-all duration-200 font-semibold"
					style="background-color: #a3be8c; focus:ring-color: #8fbcbb;"
				>
					Calculate {algorithms.find((a) => a.value === selectedAlgorithm)?.label || selectedAlgorithm}
				</button>
			</div>
		</div>
	{/if}

	<!-- Algorithm Components -->
	{#if processes.length > 0 && showResults}
		{#if selectedAlgorithm === "FCFS"}
			<FCFSComponent {processes} />
		{:else if selectedAlgorithm === "SJF"}
			<SJFComponent {processes} />
		{:else if selectedAlgorithm === "SRT"}
			<SRTComponent {processes} />
		{:else if selectedAlgorithm === "RR"}
			<RoundRobinComponent {processes} />
		{/if}
	{/if}
</div>

<style>
	:global(body) {
		background-color: #2e3440; /* Nord0 - Polar Night darkest */
		color: #d8dee9; /* Nord4 - Snow Storm lightest */
		font-family:
			"Segoe UI",
			system-ui,
			-apple-system,
			sans-serif;
	}

	:global(.nord-bg-dark) {
		background-color: #2e3440; /* Nord0 */
	}

	:global(.nord-bg-darker) {
		background-color: #3b4252; /* Nord1 */
	}

	:global(.nord-bg-darkest) {
		background-color: #434c5e; /* Nord2 */
	}

	:global(.nord-bg-light) {
		background-color: #4c566a; /* Nord3 */
	}

	:global(.nord-text-light) {
		color: #d8dee9; /* Nord4 */
	}

	:global(.nord-text-lighter) {
		color: #e5e9f0; /* Nord5 */
	}

	:global(.nord-text-lightest) {
		color: #eceff4; /* Nord6 */
	}

	:global(.nord-accent-frost-1) {
		background-color: #8fbcbb; /* Nord7 */
	}

	:global(.nord-accent-frost-2) {
		background-color: #88c0d0; /* Nord8 */
	}

	:global(.nord-accent-frost-3) {
		background-color: #81a1c1; /* Nord9 */
	}

	:global(.nord-accent-frost-4) {
		background-color: #5e81ac; /* Nord10 */
	}

	:global(.nord-accent-aurora-red) {
		background-color: #bf616a; /* Nord11 */
	}

	:global(.nord-accent-aurora-orange) {
		background-color: #d08770; /* Nord12 */
	}

	:global(.nord-accent-aurora-yellow) {
		background-color: #ebcb8b; /* Nord13 */
	}

	:global(.nord-accent-aurora-green) {
		background-color: #a3be8c; /* Nord14 */
	}

	:global(.nord-accent-aurora-purple) {
		background-color: #b48ead; /* Nord15 */
	}

	/* Hide number input spinners/arrows */
	input[type="number"]::-webkit-outer-spin-button,
	input[type="number"]::-webkit-inner-spin-button {
		-webkit-appearance: none;
		margin: 0;
	}

	/* Firefox */
	input[type="number"] {
		-moz-appearance: textfield;
	}
</style>
