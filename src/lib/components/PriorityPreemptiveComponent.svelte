<script>
	export let processes = [];

	let results = { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

	$: if (processes.length > 0) {
		calculate();
	}

	function calculate() {
		results = calculatePriorityPreemptive(processes);
	}

	function calculatePriorityPreemptive(processes) {
		if (processes.length === 0) return { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

		const processResults = [];
		const ganttChart = [];
		let currentTime = 0;

		// If no priority is specified, assign default priorities (lower PID = higher priority)
		let remainingProcesses = processes.map((p) => ({
			...p,
			priority: p.priority !== undefined ? p.priority : p.pid,
			remainingTime: p.burstTime,
			startTime: null,
		}));

		const maxTime = Math.max(...processes.map((p) => p.arrivalTime)) + processes.reduce((sum, p) => sum + p.burstTime, 0);

		while (currentTime < maxTime && remainingProcesses.some((p) => p.remainingTime > 0)) {
			// Get processes that have arrived by current time and still have remaining time
			const availableProcesses = remainingProcesses.filter((p) => p.arrivalTime <= currentTime && p.remainingTime > 0);

			if (availableProcesses.length === 0) {
				// No process available, advance time to next arrival
				const nextArrival = Math.min(...remainingProcesses.filter((p) => p.arrivalTime > currentTime && p.remainingTime > 0).map((p) => p.arrivalTime));
				if (nextArrival !== Infinity && currentTime < nextArrival) {
					ganttChart.push({
						pid: "Idle",
						start: currentTime,
						end: nextArrival,
						duration: nextArrival - currentTime,
					});
					currentTime = nextArrival;
				} else {
					break;
				}
				continue;
			}

			// Select highest priority process among available processes (lower number = higher priority)
			const highestPriorityProcess = availableProcesses.reduce((highest, current) => (current.priority < highest.priority ? current : highest));

			// Record start time for the process if it's first time executing
			if (highestPriorityProcess.startTime === null) {
				highestPriorityProcess.startTime = currentTime;
			}

			// Execute for 1 time unit
			const executionStart = currentTime;
			currentTime += 1;
			highestPriorityProcess.remainingTime -= 1;

			// Add to Gantt chart (merge with previous segment if same process)
			if (ganttChart.length > 0 && ganttChart[ganttChart.length - 1].pid === highestPriorityProcess.pid) {
				ganttChart[ganttChart.length - 1].end = currentTime;
				ganttChart[ganttChart.length - 1].duration += 1;
			} else {
				ganttChart.push({
					pid: highestPriorityProcess.pid,
					start: executionStart,
					end: currentTime,
					duration: 1,
					priority: highestPriorityProcess.priority,
				});
			}

			// If process completed, calculate its metrics
			if (highestPriorityProcess.remainingTime === 0) {
				const completionTime = currentTime;
				const turnaroundTime = completionTime - highestPriorityProcess.arrivalTime;
				const waitingTime = turnaroundTime - highestPriorityProcess.burstTime;

				processResults.push({
					...highestPriorityProcess,
					completionTime,
					turnaroundTime,
					waitingTime,
					remainingTime: undefined, // Remove this from final results
					startTime: undefined, // Remove this from final results
				});
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
	<h2 class="text-2xl font-semibold mb-6 text-indigo-600">Priority Scheduling (Preemptive)</h2>

	<!-- Gantt Chart -->
	<div class="mb-8">
		<h3 class="text-lg font-semibold mb-4">Gantt Chart</h3>
		<div class="rounded-md overflow-hidden">
			<div class="flex">
				{#each results.ganttChart as segment}
					<div class="flex flex-col items-center justify-center text-white font-medium text-sm border-r border-gray-300 last:border-r-0" class:bg-indigo-500={segment.pid !== "Idle"} class:bg-gray-400={segment.pid === "Idle"} style="width: {Math.max(segment.duration * 60, 80)}px; height: 60px;" title="Process {segment.pid}: {segment.start} - {segment.end} (Duration: {segment.duration}, Priority: {segment.priority || 'N/A'})">
						<div>P{segment.pid}</div>
						{#if segment.priority !== undefined && segment.pid !== "Idle"}
							<div class="text-xs opacity-90">Pri: {segment.priority}</div>
						{/if}
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
				<thead class="bg-indigo-50">
					<tr>
						<th class="border border-gray-300 px-4 py-3 text-left">PID</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Priority</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Arrival Time</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Burst Time</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Completion Time</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Turnaround Time</th>
						<th class="border border-gray-300 px-4 py-3 text-left">Waiting Time</th>
					</tr>
				</thead>
				<tbody>
					{#each results.processResults as process}
						<tr class="hover:bg-gray-50">
							<td class="border border-gray-300 px-4 py-2 text-center font-medium">P{process.pid}</td>
							<td class="border border-gray-300 px-4 py-2 text-center">{process.priority}</td>
							<td class="border border-gray-300 px-4 py-2 text-center">{process.arrivalTime}</td>
							<td class="border border-gray-300 px-4 py-2 text-center">{process.burstTime}</td>
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
	<div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
		<div class="bg-indigo-50 rounded-lg p-4 border border-indigo-200">
			<h4 class="font-semibold text-indigo-800 mb-2">Average Turnaround Time</h4>
			<p class="text-2xl font-bold text-indigo-600">{results.avgTurnaroundTime}</p>
		</div>
		<div class="bg-blue-50 rounded-lg p-4 border border-blue-200">
			<h4 class="font-semibold text-blue-800 mb-2">Average Waiting Time</h4>
			<p class="text-2xl font-bold text-blue-600">{results.avgWaitingTime}</p>
		</div>
	</div>

	<!-- Formulas Section -->
	<div class="rounded-lg p-4" style="background-color: #434c5e; border: 1px solid #4c566a;">
		<h3 class="text-lg font-semibold mb-4 text-white">Formulas</h3>
		<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
			<div>
				<h4 class="text-md font-semibold mb-2 text-white">Priority Preemptive Specific:</h4>
				<div class="space-y-2">
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">Select highest priority (lower number = higher priority)</div>
					</div>
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">Preemptive algorithm (can interrupt)</div>
					</div>
				</div>
			</div>
			<div>
				<h4 class="text-md font-semibold mb-2 text-white">General Scheduling:</h4>
				<div class="space-y-2">
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-green-400 text-sm">Completion Time = When process finishes</div>
					</div>
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-green-400 text-sm">Turnaround Time = Completion Time - Arrival Time</div>
					</div>
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-green-400 text-sm">Waiting Time = Turnaround Time - Burst Time</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
