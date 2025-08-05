<script>
	export let processes = [];

	let timeQuantum = 2;
	let results = { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

	$: if (processes.length > 0) {
		calculate();
	}

	function calculate() {
		results = calculateRoundRobin(processes, timeQuantum);
	}

	function calculateRoundRobin(processes, quantum) {
		if (processes.length === 0) return { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

		const processResults = [];
		const ganttChart = [];
		let currentTime = 0;
		let queue = [];
		let remainingProcesses = processes.map((p) => ({ ...p, remainingTime: p.burstTime }));
		let completedCount = 0;
		let processIndex = 0;

		// Sort processes by arrival time
		remainingProcesses.sort((a, b) => a.arrivalTime - b.arrivalTime);

		while (completedCount < processes.length) {
			// Add newly arrived processes to queue
			while (processIndex < remainingProcesses.length && remainingProcesses[processIndex].arrivalTime <= currentTime) {
				if (remainingProcesses[processIndex].remainingTime > 0) {
					queue.push(remainingProcesses[processIndex]);
				}
				processIndex++;
			}

			if (queue.length === 0) {
				// No process in queue, advance time to next arrival
				if (processIndex < remainingProcesses.length) {
					const nextArrival = remainingProcesses[processIndex].arrivalTime;
					ganttChart.push({
						pid: "Idle",
						start: currentTime,
						end: nextArrival,
						duration: nextArrival - currentTime,
					});
					currentTime = nextArrival;
				}
				continue;
			}

			// Get process from front of queue
			const currentProcess = queue.shift();
			const executionTime = Math.min(currentProcess.remainingTime, quantum);

			// Execute process
			ganttChart.push({
				pid: currentProcess.pid,
				start: currentTime,
				end: currentTime + executionTime,
				duration: executionTime,
			});

			currentProcess.remainingTime -= executionTime;
			currentTime += executionTime;

			// Add newly arrived processes to queue (during execution)
			while (processIndex < remainingProcesses.length && remainingProcesses[processIndex].arrivalTime <= currentTime) {
				if (remainingProcesses[processIndex].remainingTime > 0) {
					queue.push(remainingProcesses[processIndex]);
				}
				processIndex++;
			}

			// If process completed
			if (currentProcess.remainingTime === 0) {
				const completionTime = currentTime;
				const turnaroundTime = completionTime - currentProcess.arrivalTime;
				const waitingTime = turnaroundTime - currentProcess.burstTime;

				processResults.push({
					...currentProcess,
					completionTime,
					turnaroundTime,
					waitingTime,
				});

				completedCount++;
			} else {
				// Process not completed, add back to queue
				queue.push(currentProcess);
			}
		}

		const avgTurnaroundTime = processResults.reduce((sum, p) => sum + p.turnaroundTime, 0) / processes.length;
		const avgWaitingTime = processResults.reduce((sum, p) => sum + p.waitingTime, 0) / processes.length;

		return {
			processResults: processResults.sort((a, b) => a.pid - b.pid),
			ganttChart,
			avgTurnaroundTime: parseFloat(avgTurnaroundTime.toFixed(2)),
			avgWaitingTime: parseFloat(avgWaitingTime.toFixed(2)),
		};
	}
</script>

<div class="bg-white rounded-lg shadow-md p-6">
	<h2 class="text-2xl font-semibold mb-6 text-orange-600">Round Robin Scheduling</h2>

	<!-- Time Quantum Input -->
	<div class="mb-6">
		<label for="quantum" class="block text-sm font-medium text-gray-700 mb-2">Time Quantum</label>
		<div class="flex items-center gap-3">
			<input id="quantum" type="number" bind:value={timeQuantum} min="1" class="w-32 p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-orange-500" />
			<button on:click={calculate} class="bg-orange-500 text-white px-4 py-2 rounded-md hover:bg-orange-600 focus:outline-none focus:ring-2 focus:ring-orange-500 transition-colors font-medium"> Go </button>
		</div>
	</div>

	<!-- Gantt Chart -->
	<div class="mb-8">
		<h3 class="text-lg font-semibold mb-4">Gantt Chart</h3>
		<div class="rounded-md overflow-hidden">
			<div class="flex">
				{#each results.ganttChart as segment}
					<div class="flex items-center justify-center text-white font-medium text-sm border-r border-gray-300 last:border-r-0" class:bg-orange-500={segment.pid !== "Idle"} class:bg-gray-400={segment.pid === "Idle"} style="width: {Math.max(segment.duration * 60, 80)}px; height: 60px;" title="Process {segment.pid}: {segment.start} - {segment.end} (Duration: {segment.duration})">
						P{segment.pid}
					</div>
				{/each}
			</div>
			<div class="flex relative" style="height: 30px;">
				{#each results.ganttChart as segment, index}
					<div class="relative" style="width: {Math.max(segment.duration * 60, 80)}px;">
						{#if index === 0}
							<div class="absolute left-0 top-2 text-sm font-medium text-gray-700">
								{segment.start}
							</div>
						{/if}
						<div class="absolute right-0 top-2 text-sm font-medium text-gray-700">
							{segment.end}
						</div>
					</div>
				{/each}
			</div>
		</div>
	</div>

	<!-- Results Table -->
	<div class="mb-6">
		<h3 class="text-lg font-semibold mb-4">Process Details</h3>
		<div class="overflow-x-auto">
			<table class="w-full border-collapse border border-gray-300">
				<thead class="bg-orange-50">
					<tr>
						<th class="border border-gray-300 px-4 py-3 text-left">PID</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Burst Time</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Arrival Time</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Completion Time</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Turnaround Time</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Waiting Time</th>
					</tr>
				</thead>
				<tbody>
					{#each results.processResults as process}
						<tr class="hover:bg-gray-50">
							<td class="border border-gray-300 px-4 py-2 text-center font-medium">P{process.pid}</td>
							<td class="border border-gray-300 px-4 py-2 text-center">{process.burstTime}</td>
							<td class="border border-gray-300 px-4 py-2 text-center">{process.arrivalTime}</td>
							<td class="border border-gray-300 px-4 py-2 text-center">{process.completionTime}</td>
							<td class="border border-gray-300 px-4 py-2 text-center">{process.turnaroundTime}</td>
							<td class="border border-gray-300 px-4 py-2 text-center">{process.waitingTime}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	</div>

	<!-- Average Times -->
	<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
		<div class="bg-green-50 rounded-lg p-4 border border-green-200">
			<h4 class="font-semibold text-green-800 mb-2">Average Turnaround Time</h4>
			<p class="text-2xl font-bold text-green-600">{results.avgTurnaroundTime}</p>
		</div>
		<div class="bg-blue-50 rounded-lg p-4 border border-blue-200">
			<h4 class="font-semibold text-blue-800 mb-2">Average Waiting Time</h4>
			<p class="text-2xl font-bold text-blue-600">{results.avgWaitingTime}</p>
		</div>
	</div>
</div>
