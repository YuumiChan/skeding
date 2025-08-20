<script>
	import FCFSComponent from "$lib/components/FCFSComponent.svelte";
	import HRRNComponent from "$lib/components/HRRNComponent.svelte";
	import MFQComponent from "$lib/components/MFQComponent.svelte";
	import MLQComponent from "$lib/components/MLQComponent.svelte";
	import PriorityNonPreemptiveComponent from "$lib/components/PriorityNonPreemptiveComponent.svelte";
	import PriorityPreemptiveComponent from "$lib/components/PriorityPreemptiveComponent.svelte";
	import RoundRobinComponent from "$lib/components/RoundRobinComponent.svelte";
	import SJFComponent from "$lib/components/SJFComponent.svelte";
	import SRTComponent from "$lib/components/SRTComponent.svelte";

	let selectedAlgorithm = "FCFS";
	let pid = "";
	let burstTime = "";
	let arrivalTime = "";
	let priority = "";
	let processes = [];
	let showResults = false;

	const algorithms = [
		{ value: "FCFS", label: "First Come First Serve (FCFS)" },
		{ value: "SJF", label: "Shortest Job First (SJF)" },
		{ value: "SRT", label: "Shortest Remaining Time (SRT)" },
		{ value: "RR", label: "Round Robin" },
		{ value: "PNP", label: "Priority (Non-Preemptive)" },
		{ value: "PP", label: "Priority (Preemptive)" },
		{ value: "HRRN", label: "Highest Response Ratio Next (HRRN)" },
		{ value: "MLQ", label: "Multiple Level Queue (MLQ)" },
		{ value: "MFQ", label: "Multilevel Feedback Queue (MFQ)" },
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
			const priorityNum = priority && priority !== "" ? parseInt(priority) : undefined;

			// Check for duplicate PID
			if (processes.some((p) => p.pid === pidNum)) {
				alert(`Process with PID ${pidNum} already exists!`);
				return;
			}

			// Validate values
			if (pidNum <= 0 || burstNum < 0 || arrivalNum < 0 || (priorityNum !== undefined && priorityNum < 0)) {
				alert("Please enter valid values (PID > 0, Burst Time >= 0, Arrival Time >= 0, Priority >= 0)");
				return;
			}

			const newProcess = {
				pid: pidNum,
				burstTime: burstNum,
				arrivalTime: arrivalNum,
			};

			// Add priority only if specified
			if (priorityNum !== undefined) {
				newProcess.priority = priorityNum;
			}

			processes = [...processes, newProcess];

			// Clear inputs
			pid = "";
			burstTime = "";
			arrivalTime = "";
			priority = "";
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
		const hasPriorityAlgorithm = selectedAlgorithm === "PNP" || selectedAlgorithm === "PP" || selectedAlgorithm === "MLQ" || selectedAlgorithm === "MFQ";
		const isMLQ = selectedAlgorithm === "MLQ";

		processes = [
			{
				pid: 1,
				burstTime: Math.floor(Math.random() * 10) + 1,
				arrivalTime: Math.floor(Math.random() * 11),
				...(hasPriorityAlgorithm && { priority: isMLQ ? Math.floor(Math.random() * 3) + 1 : Math.floor(Math.random() * 5) + 1 }),
			},
			{
				pid: 2,
				burstTime: Math.floor(Math.random() * 10) + 1,
				arrivalTime: Math.floor(Math.random() * 11),
				...(hasPriorityAlgorithm && { priority: isMLQ ? Math.floor(Math.random() * 3) + 1 : Math.floor(Math.random() * 5) + 1 }),
			},
			{
				pid: 3,
				burstTime: Math.floor(Math.random() * 10) + 1,
				arrivalTime: Math.floor(Math.random() * 11),
				...(hasPriorityAlgorithm && { priority: isMLQ ? Math.floor(Math.random() * 3) + 1 : Math.floor(Math.random() * 5) + 1 }),
			},
			{
				pid: 4,
				burstTime: Math.floor(Math.random() * 10) + 1,
				arrivalTime: Math.floor(Math.random() * 11),
				...(hasPriorityAlgorithm && { priority: isMLQ ? Math.floor(Math.random() * 3) + 1 : Math.floor(Math.random() * 5) + 1 }),
			},
		];
		showResults = false;
	}

	function removeProcess(index) {
		processes = processes.filter((_, i) => i !== index);
		showResults = false;
	}

	function updateProcess(index, field, value) {
		const newValue = parseInt(value);

		// Handle priority field specially - allow empty values
		if (field === "priority") {
			if (value === "" || value === null || value === undefined) {
				// Remove priority property if empty
				delete processes[index].priority;
			} else {
				const priorityValue = parseInt(value);
				if (priorityValue < 0) {
					alert("Priority cannot be negative");
					return;
				}
				processes[index].priority = priorityValue;
			}
		} else {
			const numValue = newValue || 0;

			// Validate PID uniqueness when updating PID
			if (field === "pid") {
				if (processes.some((p, i) => i !== index && p.pid === numValue)) {
					alert(`Process with PID ${numValue} already exists!`);
					return;
				}
				if (numValue <= 0) {
					alert("PID must be greater than 0");
					return;
				}
			}

			// Validate burst time
			if (field === "burstTime" && numValue < 0) {
				alert("Burst time cannot be negative");
				return;
			}

			// Validate arrival time
			if (field === "arrivalTime" && numValue < 0) {
				alert("Arrival time cannot be negative");
				return;
			}

			processes[index][field] = numValue;
		}

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
	<div class="bg-white rounded-lg shadow-md p-6 mb-6">
		<h2 class="text-xl font-semibold mb-4">Select Scheduling Algorithm</h2>
		<select bind:value={selectedAlgorithm} class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
			{#each algorithms as algorithm}
				<option value={algorithm.value} style="background-color: #434c5e; color: #d8dee9;">{algorithm.label}</option>
			{/each}
		</select>
	</div>

	<!-- Process Input Form -->
	<div class="rounded-lg shadow-lg p-6 mb-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
		<h2 class="text-xl font-semibold mb-4 text-white">Add Process</h2>
		<div class="grid grid-cols-1 md:grid-cols-5 gap-4">
			<div>
				<label for="pid" class="block text-sm font-medium text-gray-700 mb-2">Process ID</label>
				<input id="pid" type="number" bind:value={pid} placeholder="Auto-assign if empty" min="1" on:keypress={handleKeyPress} class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" />
			</div>
			<div>
				<label for="arrival" class="block text-sm font-medium text-gray-700 mb-2">Arrival Time</label>
				<input id="arrival" type="number" bind:value={arrivalTime} placeholder="Default: 0" min="0" on:keypress={handleKeyPress} class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" />
			</div>
			<div>
				<label for="burst" class="block text-sm font-medium text-gray-700 mb-2">Burst Time <span class="text-red-500">*</span></label>
				<input id="burst" type="number" bind:value={burstTime} placeholder="Enter burst time" min="1" on:keypress={handleKeyPress} class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" />
			</div>
			<div>
				<label for="priority" class="block text-sm font-medium text-gray-700 mb-2"
					>Priority {#if selectedAlgorithm === "PNP" || selectedAlgorithm === "PP"}<span class="text-red-500">*</span>{/if}</label
				>
				<input id="priority" type="number" bind:value={priority} placeholder={selectedAlgorithm === "PNP" || selectedAlgorithm === "PP" ? "Lower = Higher Priority" : "Optional"} min="0" on:keypress={handleKeyPress} class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" />
			</div>
			<div class="flex items-end gap-2">
				<button on:click={addProcess} class="flex-1 bg-blue-500 text-white p-3 rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors"> Add Process </button>
				<button on:click={addExampleData} class="bg-gray-500 text-white px-4 py-3 rounded-md hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-gray-500 transition-colors text-sm" title="Add example processes"> Example </button>
			</div>
		</div>
		<p class="text-sm text-gray-600 mt-3">
			💡 <strong>Tips:</strong> Press Enter to add process • PID auto-assigns if empty • Arrival time defaults to 0 • Priority required for Priority algorithms • Only Burst Time is always required
		</p>
	</div>

	<!-- Process List -->
	{#if processes.length > 0}
		<div class="rounded-lg shadow-lg p-6 mb-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
			<div class="flex justify-between items-center mb-4">
				<h2 class="text-xl font-semibold text-white">Current Processes ({processes.length})</h2>
				<div class="flex gap-2">
					<button on:click={clearProcesses} class="bg-red-500 text-white px-4 py-2 rounded-md hover:bg-red-600 transition-colors"> Clear All </button>
				</div>
			</div>
			<div class="overflow-x-auto">
				<table class="w-full border-collapse rounded-lg overflow-hidden" style="border: 1px solid #4c566a;">
					<thead style="background-color: #434c5e;">
						<tr>
							<th class="border border-gray-300 px-4 py-2">PID</th>
							<th class="border border-gray-300 px-4 py-2">Arrival Time</th>
							<th class="border border-gray-300 px-4 py-2">Burst Time</th>
							{#if selectedAlgorithm === "PNP" || selectedAlgorithm === "PP" || selectedAlgorithm === "MFQ" || selectedAlgorithm === "MLQ"}
								<th class="border border-gray-300 px-4 py-2">Priority</th>
							{/if}
							<th class="border border-gray-300 px-4 py-2">Action</th>
						</tr>
					</thead>
					<tbody>
						{#each processes as process, index}
							<tr class="hover:bg-gray-50">
								<td class="border border-gray-300 px-2 py-2 text-center">
									<input type="number" value={process.pid} on:input={(e) => updateProcess(index, "pid", e.target.value)} class="w-full text-center border-0 bg-transparent focus:outline-none focus:bg-blue-50 rounded px-2 py-1" min="1" />
								</td>
								<td class="border border-gray-300 px-2 py-2 text-center">
									<input type="number" value={process.arrivalTime} on:input={(e) => updateProcess(index, "arrivalTime", e.target.value)} class="w-full text-center border-0 bg-transparent focus:outline-none focus:bg-blue-50 rounded px-2 py-1" min="0" />
								</td>
								<td class="border border-gray-300 px-2 py-2 text-center">
									<input type="number" value={process.burstTime} on:input={(e) => updateProcess(index, "burstTime", e.target.value)} class="w-full text-center border-0 bg-transparent focus:outline-none focus:bg-blue-50 rounded px-2 py-1" min="0" />
								</td>
								{#if selectedAlgorithm === "PNP" || selectedAlgorithm === "PP" || selectedAlgorithm === "MFQ" || selectedAlgorithm === "MLQ"}
									<td class="border border-gray-300 px-2 py-2 text-center">
										<input type="number" value={process.priority || ""} on:input={(e) => updateProcess(index, "priority", e.target.value)} class="w-full text-center border-0 bg-transparent focus:outline-none focus:bg-blue-50 rounded px-2 py-1" min="0" placeholder="Optional" />
									</td>
								{/if}
								<td class="border border-gray-300 px-4 py-2 text-center">
									<button on:click={() => removeProcess(index)} class="bg-red-500 text-white px-3 py-1 rounded text-sm hover:bg-red-600 transition-colors"> Remove </button>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
			<p class="text-sm mt-2" style="color: #d8dee9;">💡 Tip: Click on any value in the table to edit it directly</p>

			<!-- Calculate Button -->
			<div class="mt-4 text-center">
				<button on:click={calculateResults} class="bg-green-500 text-white px-6 py-3 rounded-md hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500 transition-colors font-semibold">
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
		{:else if selectedAlgorithm === "PNP"}
			<PriorityNonPreemptiveComponent {processes} />
		{:else if selectedAlgorithm === "PP"}
			<PriorityPreemptiveComponent {processes} />
		{:else if selectedAlgorithm === "HRRN"}
			<HRRNComponent {processes} />
		{:else if selectedAlgorithm === "MLQ"}
			<MLQComponent {processes} />
		{:else if selectedAlgorithm === "MFQ"}
			<MFQComponent {processes} />
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

	/* Nord Color Palette Variables */
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

	/* Frost colors */
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

	/* Aurora colors */
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

	/* Override default Tailwind classes with Nord colors */
	:global(.bg-white) {
		background-color: #3b4252 !important; /* Nord1 */
		color: #d8dee9 !important; /* Nord4 */
	}

	:global(.text-gray-700) {
		color: #d8dee9 !important; /* Nord4 */
	}

	:global(.text-gray-600) {
		color: #e5e9f0 !important; /* Nord5 */
	}

	:global(.text-sm) {
		color: #e5e9f0 !important; /* Nord5 */
	}

	/* Input and form elements */
	:global(input) {
		background-color: #434c5e !important; /* Nord2 */
		border-color: #4c566a !important; /* Nord3 */
		color: #d8dee9 !important; /* Nord4 */
	}

	:global(input:focus) {
		background-color: #434c5e !important; /* Nord2 */
		border-color: #88c0d0 !important; /* Nord8 */
		box-shadow: 0 0 0 2px rgba(136, 192, 208, 0.3) !important;
		outline: none !important;
	}

	:global(input::placeholder) {
		color: #4c566a !important; /* Nord3 */
	}

	:global(input:hover) {
		background-color: #3b4252 !important; /* Nord1 */
		border-color: #4c566a !important; /* Nord3 */
	}

	/* Select dropdown */
	:global(select) {
		background-color: #434c5e !important; /* Nord2 */
		border-color: #4c566a !important; /* Nord3 */
		color: #d8dee9 !important; /* Nord4 */
	}

	:global(select:focus) {
		background-color: #434c5e !important; /* Nord2 */
		border-color: #88c0d0 !important; /* Nord8 */
		box-shadow: 0 0 0 2px rgba(136, 192, 208, 0.3) !important;
		outline: none !important;
	}

	:global(select:hover) {
		background-color: #3b4252 !important; /* Nord1 */
	}

	/* Button styling with Nord colors */
	:global(.bg-blue-500) {
		background-color: #5e81ac !important; /* Nord10 */
	}

	:global(.bg-blue-500:hover) {
		background-color: #81a1c1 !important; /* Nord9 */
	}

	:global(.bg-green-500) {
		background-color: #a3be8c !important; /* Nord14 */
	}

	:global(.bg-green-500:hover) {
		background-color: #93a876 !important;
	}

	:global(.bg-red-500) {
		background-color: #bf616a !important; /* Nord11 */
	}

	:global(.bg-red-500:hover) {
		background-color: #a54e58 !important;
	}

	:global(.bg-gray-500) {
		background-color: #4c566a !important; /* Nord3 */
	}

	:global(.bg-gray-500:hover) {
		background-color: #434c5e !important; /* Nord2 */
	}

	/* Table styling */
	:global(table) {
		background-color: #3b4252 !important; /* Nord1 */
	}

	:global(thead) {
		background-color: #434c5e !important; /* Nord2 */
	}

	:global(.bg-gray-50) {
		background-color: #434c5e !important; /* Nord2 */
	}

	:global(tr:hover) {
		background-color: #434c5e !important; /* Nord2 */
	}

	:global(.hover\\:bg-gray-50:hover) {
		background-color: #434c5e !important; /* Nord2 */
	}

	:global(th),
	:global(td) {
		border-color: #4c566a !important; /* Nord3 */
		color: #d8dee9 !important; /* Nord4 */
	}

	:global(.border-gray-300) {
		border-color: #4c566a !important; /* Nord3 */
	}

	/* Algorithm specific colors */
	:global(.text-green-600) {
		color: #a3be8c !important; /* Nord14 */
	}

	:global(.text-blue-600) {
		color: #88c0d0 !important; /* Nord8 */
	}

	:global(.text-purple-600) {
		color: #b48ead !important; /* Nord15 */
	}

	:global(.text-indigo-600) {
		color: #5e81ac !important; /* Nord10 */
	}

	:global(.text-yellow-600) {
		color: #ebcb8b !important; /* Nord13 */
	}

	:global(.text-red-500) {
		color: #bf616a !important; /* Nord11 */
	}

	/* Background colors for algorithm components */
	:global(.bg-green-50) {
		background-color: rgba(163, 190, 140, 0.1) !important;
		border-radius: 0.5rem;
	}

	:global(.bg-blue-50) {
		background-color: rgba(136, 192, 208, 0.1) !important;
		border-radius: 0.5rem;
	}

	:global(.bg-purple-50) {
		background-color: rgba(180, 142, 173, 0.1) !important;
		border-radius: 0.5rem;
	}

	:global(.bg-indigo-50) {
		background-color: rgba(94, 129, 172, 0.1) !important;
		border-radius: 0.5rem;
	}

	:global(.bg-yellow-50) {
		background-color: rgba(235, 203, 139, 0.1) !important;
		border-radius: 0.5rem;
	}

	/* Border colors for algorithm components */
	:global(.border-green-200) {
		border-color: rgba(163, 190, 140, 0.3) !important;
	}

	:global(.border-blue-200) {
		border-color: rgba(136, 192, 208, 0.3) !important;
	}

	:global(.border-purple-200) {
		border-color: rgba(180, 142, 173, 0.3) !important;
	}

	:global(.border-indigo-200) {
		border-color: rgba(94, 129, 172, 0.3) !important;
	}

	:global(.border-yellow-200) {
		border-color: rgba(235, 203, 139, 0.3) !important;
	}

	/* Text colors for algorithm components */
	:global(.text-green-800) {
		color: #a3be8c !important; /* Nord14 */
	}

	:global(.text-blue-800) {
		color: #88c0d0 !important; /* Nord8 */
	}

	:global(.text-purple-800) {
		color: #b48ead !important; /* Nord15 */
	}

	:global(.text-indigo-800) {
		color: #5e81ac !important; /* Nord10 */
	}

	:global(.text-yellow-800) {
		color: #ebcb8b !important; /* Nord13 */
	}

	/* Gantt chart colors */
	:global(.bg-purple-500) {
		background-color: #b48ead !important; /* Nord15 */
	}

	:global(.bg-indigo-500) {
		background-color: #5e81ac !important; /* Nord10 */
	}

	:global(.bg-yellow-500) {
		background-color: #ebcb8b !important; /* Nord13 */
	}

	:global(.bg-gray-400) {
		background-color: #4c566a !important; /* Nord3 */
	}

	/* Shadow styling with Nord colors */
	:global(.shadow-md) {
		box-shadow:
			0 4px 6px -1px rgba(46, 52, 64, 0.4),
			0 2px 4px -1px rgba(46, 52, 64, 0.3) !important;
	}

	/* Input focus styling for table cells */
	:global(.focus\\:bg-blue-50:focus) {
		background-color: rgba(136, 192, 208, 0.15) !important;
	}

	/* Scrollbar styling with Nord colors */
	:global(::-webkit-scrollbar) {
		width: 8px;
		height: 8px;
	}

	:global(::-webkit-scrollbar-track) {
		background: #3b4252; /* Nord1 */
	}

	:global(::-webkit-scrollbar-thumb) {
		background: #4c566a; /* Nord3 */
		border-radius: 4px;
	}

	:global(::-webkit-scrollbar-thumb:hover) {
		background: #434c5e; /* Nord2 */
	}

	/* Ring colors for focus states */
	:global(.focus\\:ring-blue-500:focus) {
		box-shadow: 0 0 0 2px rgba(136, 192, 208, 0.3) !important;
	}

	:global(.focus\\:ring-green-500:focus) {
		box-shadow: 0 0 0 2px rgba(163, 190, 140, 0.3) !important;
	}

	:global(.focus\\:ring-gray-500:focus) {
		box-shadow: 0 0 0 2px rgba(76, 86, 106, 0.3) !important;
	}

	/* Hide number input spinners/arrows */
	:global(input[type="number"]::-webkit-outer-spin-button),
	:global(input[type="number"]::-webkit-inner-spin-button) {
		-webkit-appearance: none;
		margin: 0;
	}

	/* Firefox */
	:global(input[type="number"]) {
		-moz-appearance: textfield;
	}
</style>
