<script>
	export let processes = [];

	$: results = calculateSJF(processes);

	function calculateSJF(processes) {
		if (processes.length === 0) return { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

		const processResults = [];
		const ganttChart = [];
		let currentTime = 0;
		let remainingProcesses = [...processes];

		while (remainingProcesses.length > 0) {
			// Get processes that have arrived by current time
			const availableProcesses = remainingProcesses.filter((p) => p.arrivalTime <= currentTime);

			if (availableProcesses.length === 0) {
				// No process available, advance time to next arrival
				const nextArrival = Math.min(...remainingProcesses.map((p) => p.arrivalTime));
				if (currentTime < nextArrival) {
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

			// Select shortest job among available processes
			const shortestJob = availableProcesses.reduce((shortest, current) => (current.burstTime < shortest.burstTime ? current : shortest));

			const startTime = currentTime;
			const completionTime = currentTime + shortestJob.burstTime;
			const turnaroundTime = completionTime - shortestJob.arrivalTime;
			const waitingTime = turnaroundTime - shortestJob.burstTime;

			processResults.push({
				...shortestJob,
				completionTime,
				turnaroundTime,
				waitingTime,
			});

			ganttChart.push({
				pid: shortestJob.pid,
				start: startTime,
				end: completionTime,
				duration: shortestJob.burstTime,
			});

			currentTime = completionTime;
			remainingProcesses = remainingProcesses.filter((p) => p.pid !== shortestJob.pid);
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
	<h2 class="text-2xl font-semibold mb-6 text-green-600">Shortest Job First (SJF) Scheduling</h2>

	<!-- Gantt Chart -->
	<div class="mb-8">
		<h3 class="text-lg font-semibold mb-4">Gantt Chart</h3>
		<div class="border border-gray-300 rounded-md overflow-hidden">
			<div class="flex">
				{#each results.ganttChart as segment}
					<div
						class="flex items-center justify-center text-white font-medium text-sm border-r border-gray-300 last:border-r-0"
						class:bg-green-500={segment.pid !== "Idle"}
						class:bg-gray-400={segment.pid === "Idle"}
						style="width: {Math.max(segment.duration * 40, 60)}px; height: 50px;"
						title="Process {segment.pid}: {segment.start} - {segment.end} (Duration: {segment.duration})"
					>
						P{segment.pid}
					</div>
				{/each}
			</div>
			<div class="flex border-t border-gray-300">
				{#each results.ganttChart as segment}
					<div
						class="text-xs text-center border-r border-gray-300 last:border-r-0 py-1"
						style="width: {Math.max(segment.duration * 40, 60)}px;"
					>
						{segment.start}
					</div>
				{/each}
				<div class="text-xs py-1 px-2">
					{results.ganttChart.length > 0 ? results.ganttChart[results.ganttChart.length - 1].end : 0}
				</div>
			</div>
		</div>
	</div>

	<!-- Results Table -->
	<div class="mb-6">
		<h3 class="text-lg font-semibold mb-4">Process Details</h3>
		<div class="overflow-x-auto">
			<table class="w-full border-collapse border border-gray-300">
				<thead class="bg-green-50">
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
