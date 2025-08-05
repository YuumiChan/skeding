<script>
	export let processes = [];

	let results = { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

	$: if (processes.length > 0) {
		calculate();
	}

	function calculate() {
		const sortedProcesses = [...processes].sort((a, b) => a.arrivalTime - b.arrivalTime);
		results = calculateFCFS(sortedProcesses);
	}

	function calculateFCFS(processes) {
		if (processes.length === 0) return { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

		const processResults = [];
		const ganttChart = [];
		let currentTime = 0;

		for (let i = 0; i < processes.length; i++) {
			const process = processes[i];

			// If current time is less than arrival time, wait
			if (currentTime < process.arrivalTime) {
				if (currentTime !== process.arrivalTime) {
					ganttChart.push({
						pid: "Idle",
						start: currentTime,
						end: process.arrivalTime,
						duration: process.arrivalTime - currentTime,
					});
				}
				currentTime = process.arrivalTime;
			}

			const startTime = currentTime;
			const completionTime = currentTime + process.burstTime;
			const turnaroundTime = completionTime - process.arrivalTime;
			const waitingTime = turnaroundTime - process.burstTime;

			processResults.push({
				...process,
				completionTime,
				turnaroundTime,
				waitingTime,
			});

			ganttChart.push({
				pid: process.pid,
				start: startTime,
				end: completionTime,
				duration: process.burstTime,
			});

			currentTime = completionTime;
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
	<h2 class="text-2xl font-semibold mb-6 text-blue-600">First Come First Serve (FCFS) Scheduling</h2>

	<!-- Gantt Chart -->
	<div class="mb-8">
		<h3 class="text-lg font-semibold mb-4">Gantt Chart</h3>
		<div class="rounded-md overflow-hidden">
			<div class="flex">
				{#each results.ganttChart as segment}
					<div class="flex items-center justify-center text-white font-medium text-sm border-r border-gray-300 last:border-r-0" class:bg-blue-500={segment.pid !== "Idle"} class:bg-gray-400={segment.pid === "Idle"} style="width: {Math.max(segment.duration * 60, 80)}px; height: 60px;" title="Process {segment.pid}: {segment.start} - {segment.end} (Duration: {segment.duration})">
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
				<thead class="bg-blue-50">
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
