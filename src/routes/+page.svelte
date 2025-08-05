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
		if (pid && burstTime && arrivalTime) {
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
			alert("Please fill all fields");
		}
	}

	function clearProcesses() {
		processes = [];
		showResults = false;
	}

	function addExampleData() {
		processes = [
			{ pid: 1, burstTime: Math.floor(Math.random() * 10) + 1, arrivalTime: Math.floor(Math.random() * 5) },
			{ pid: 2, burstTime: Math.floor(Math.random() * 10) + 1, arrivalTime: Math.floor(Math.random() * 5) },
			{ pid: 3, burstTime: Math.floor(Math.random() * 10) + 1, arrivalTime: Math.floor(Math.random() * 5) },
			{ pid: 4, burstTime: Math.floor(Math.random() * 10) + 1, arrivalTime: Math.floor(Math.random() * 5) },
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
	<h1 class="text-3xl font-bold text-center mb-8">CPU Scheduling Algorithms</h1>

	<!-- Algorithm Selection -->
	<div class="bg-white rounded-lg shadow-md p-6 mb-6">
		<h2 class="text-xl font-semibold mb-4">Select Scheduling Algorithm</h2>
		<select bind:value={selectedAlgorithm} class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
			{#each algorithms as algorithm}
				<option value={algorithm.value}>{algorithm.label}</option>
			{/each}
		</select>
	</div>

	<!-- Process Input Form -->
	<div class="bg-white rounded-lg shadow-md p-6 mb-6">
		<h2 class="text-xl font-semibold mb-4">Add Process</h2>
		<div class="grid grid-cols-1 md:grid-cols-4 gap-4">
			<div>
				<label for="pid" class="block text-sm font-medium text-gray-700 mb-2">Process ID</label>
				<input id="pid" type="number" bind:value={pid} placeholder="Enter PID" min="1" class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" />
			</div>
			<div>
				<label for="burst" class="block text-sm font-medium text-gray-700 mb-2">Burst Time</label>
				<input id="burst" type="number" bind:value={burstTime} placeholder="Enter burst time" min="0" class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" />
			</div>
			<div>
				<label for="arrival" class="block text-sm font-medium text-gray-700 mb-2">Arrival Time</label>
				<input id="arrival" type="number" bind:value={arrivalTime} placeholder="Enter arrival time" min="0" class="w-full p-3 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" />
			</div>
			<div class="flex items-end gap-2">
				<button on:click={addProcess} class="flex-1 bg-blue-500 text-white p-3 rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors"> Add Process </button>
				<button on:click={addExampleData} class="bg-gray-500 text-white px-4 py-3 rounded-md hover:bg-gray-600 focus:outline-none focus:ring-2 focus:ring-gray-500 transition-colors text-sm" title="Add example processes"> Example </button>
			</div>
		</div>
	</div>

	<!-- Process List -->
	{#if processes.length > 0}
		<div class="bg-white rounded-lg shadow-md p-6 mb-6">
			<div class="flex justify-between items-center mb-4">
				<h2 class="text-xl font-semibold">Current Processes ({processes.length})</h2>
				<div class="flex gap-2">
					<button on:click={addExampleData} class="bg-green-500 text-white px-4 py-2 rounded-md hover:bg-green-600 transition-colors text-sm" disabled={processes.length > 0} class:opacity-50={processes.length > 0} class:cursor-not-allowed={processes.length > 0}> Load Example </button>
					<button on:click={clearProcesses} class="bg-red-500 text-white px-4 py-2 rounded-md hover:bg-red-600 transition-colors"> Clear All </button>
				</div>
			</div>
			<div class="overflow-x-auto">
				<table class="w-full border-collapse border border-gray-300">
					<thead class="bg-gray-50">
						<tr>
							<th class="border border-gray-300 px-4 py-2">PID</th>
							<th class="border border-gray-300 px-4 py-2">Burst Time</th>
							<th class="border border-gray-300 px-4 py-2">Arrival Time</th>
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
									<input type="number" value={process.burstTime} on:input={(e) => updateProcess(index, "burstTime", e.target.value)} class="w-full text-center border-0 bg-transparent focus:outline-none focus:bg-blue-50 rounded px-2 py-1" min="0" />
								</td>
								<td class="border border-gray-300 px-2 py-2 text-center">
									<input type="number" value={process.arrivalTime} on:input={(e) => updateProcess(index, "arrivalTime", e.target.value)} class="w-full text-center border-0 bg-transparent focus:outline-none focus:bg-blue-50 rounded px-2 py-1" min="0" />
								</td>
								<td class="border border-gray-300 px-4 py-2 text-center">
									<button on:click={() => removeProcess(index)} class="bg-red-500 text-white px-3 py-1 rounded text-sm hover:bg-red-600 transition-colors"> Remove </button>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
			<p class="text-sm text-gray-600 mt-2">💡 Tip: Click on any value in the table to edit it directly</p>

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
		{/if}
	{/if}
</div>

<style>
	:global(body) {
		background-color: #f3f4f6;
	}
</style>
