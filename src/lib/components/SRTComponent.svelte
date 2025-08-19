<script>
	export let processes = [];

	let results = { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

	$: if (processes.length > 0) {
		calculate();
	}

	function calculate() {
		results = calculateSRT(processes);
	}

	function calculateSRT(processes) {
		if (processes.length === 0) return { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

		const processResults = [];
		const ganttChart = [];
		let currentTime = 0;
		let remainingProcesses = processes.map((p) => ({ ...p, remainingTime: p.burstTime }));
		let completedCount = 0;

		while (completedCount < processes.length) {
			// Get processes that have arrived by current time and still have remaining time
			const availableProcesses = remainingProcesses.filter((p) => p.arrivalTime <= currentTime && p.remainingTime > 0);

			if (availableProcesses.length === 0) {
				// No process available, advance time to next arrival
				const nextArrival = Math.min(...remainingProcesses.filter((p) => p.remainingTime > 0).map((p) => p.arrivalTime));

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

			// Select process with shortest remaining time
			const shortestProcess = availableProcesses.reduce((shortest, current) => (current.remainingTime < shortest.remainingTime ? current : shortest));

			// Check if we need to preempt (if there's a new process that arrives during execution)
			const nextEventTime = getNextEventTime(currentTime, remainingProcesses, shortestProcess);
			const executionTime = Math.min(shortestProcess.remainingTime, nextEventTime - currentTime);

			if (executionTime > 0) {
				// Execute the process
				ganttChart.push({
					pid: shortestProcess.pid,
					start: currentTime,
					end: currentTime + executionTime,
					duration: executionTime,
				});

				shortestProcess.remainingTime -= executionTime;
				currentTime += executionTime;

				// If process completed
				if (shortestProcess.remainingTime === 0) {
					const completionTime = currentTime;
					const turnaroundTime = completionTime - shortestProcess.arrivalTime;
					const waitingTime = turnaroundTime - shortestProcess.burstTime;

					processResults.push({
						...shortestProcess,
						completionTime,
						turnaroundTime,
						waitingTime,
					});

					completedCount++;
				}
			}
		}

		// Merge consecutive segments of the same process in Gantt chart
		const mergedGanttChart = mergeGanttSegments(ganttChart);

		const avgTurnaroundTime = processResults.reduce((sum, p) => sum + p.turnaroundTime, 0) / processes.length;
		const avgWaitingTime = processResults.reduce((sum, p) => sum + p.waitingTime, 0) / processes.length;

		return {
			processResults: processResults.sort((a, b) => a.pid - b.pid),
			ganttChart: mergedGanttChart,
			avgTurnaroundTime: parseFloat(avgTurnaroundTime.toFixed(2)),
			avgWaitingTime: parseFloat(avgWaitingTime.toFixed(2)),
		};
	}

	function getNextEventTime(currentTime, remainingProcesses, currentProcess) {
		// Find the next arrival time that could preempt current process
		const futureArrivals = remainingProcesses.filter((p) => p.arrivalTime > currentTime && p.remainingTime > 0).map((p) => p.arrivalTime);

		if (futureArrivals.length === 0) {
			return currentTime + currentProcess.remainingTime;
		}

		const nextArrival = Math.min(...futureArrivals);
		return Math.min(nextArrival, currentTime + currentProcess.remainingTime);
	}

	function mergeGanttSegments(ganttChart) {
		if (ganttChart.length === 0) return ganttChart;

		const merged = [ganttChart[0]];

		for (let i = 1; i < ganttChart.length; i++) {
			const current = ganttChart[i];
			const last = merged[merged.length - 1];

			if (current.pid === last.pid && current.start === last.end) {
				// Merge with previous segment
				last.end = current.end;
				last.duration = last.end - last.start;
			} else {
				merged.push(current);
			}
		}

		return merged;
	}
</script>

<div class="rounded-lg shadow-lg p-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
	<h2 class="text-2xl font-semibold mb-6 text-white">Shortest Remaining Time (SRT) Scheduling</h2>

	<!-- Gantt Chart -->
	<div class="mb-8">
		<h3 class="text-lg font-semibold mb-4 text-white">Gantt Chart</h3>
		<div class="rounded-md overflow-hidden" style="background-color: #434c5e;">
			<div class="flex">
				{#each results.ganttChart as segment}
					<div class="flex items-center justify-center text-white font-medium text-sm border-r last:border-r-0" style="width: {Math.max(segment.duration * 60, 80)}px; height: 60px; background-color: {segment.pid !== 'Idle' ? '#b48ead' : '#4c566a'}; border-color: #2e3440;" title="Process {segment.pid}: {segment.start} - {segment.end} (Duration: {segment.duration})">
						P{segment.pid}
					</div>
				{/each}
			</div>
			<div class="flex relative" style="height: 30px;">
				{#each results.ganttChart as segment, index}
					<div class="relative" style="width: {Math.max(segment.duration * 60, 80)}px;">
						{#if index === 0}
							<div class="absolute left-0 top-2 text-sm font-medium text-white">
								{segment.start}
							</div>
						{/if}
						<div class="absolute right-0 top-2 text-sm font-medium text-white">
							{segment.end}
						</div>
					</div>
				{/each}
			</div>
		</div>
	</div>

	<!-- Results Table -->
	<div class="mb-6">
		<h3 class="text-lg font-semibold mb-4 text-white">Process Details</h3>
		<div class="overflow-x-auto">
			<table class="w-full border-collapse rounded-lg overflow-hidden" style="border: 1px solid #4c566a;">
				<thead style="background-color: #434c5e;">
					<tr>
						<th class="px-4 py-3 text-left text-white font-semibold" style="border: 1px solid #4c566a;">PID</th>
						<th class="px-4 py-3 text-left text-white font-semibold" style="border: 1px solid #4c566a;">Burst Time</th>
						<th class="px-4 py-3 text-left text-white font-semibold" style="border: 1px solid #4c566a;">Arrival Time</th>
						<th class="px-4 py-3 text-left text-white font-semibold" style="border: 1px solid #4c566a;">Completion Time</th>
						<th class="px-4 py-3 text-left text-white font-semibold" style="border: 1px solid #4c566a;">Turnaround Time</th>
						<th class="px-4 py-3 text-left text-white font-semibold" style="border: 1px solid #4c566a;">Waiting Time</th>
					</tr>
				</thead>
				<tbody>
					{#each results.processResults as process}
						<tr class="hover:opacity-90 transition-all duration-200" style="background-color: #2e3440;">
							<td class="px-4 py-2 text-center font-medium text-white" style="border: 1px solid #4c566a;">P{process.pid}</td>
							<td class="px-4 py-2 text-center text-white" style="border: 1px solid #4c566a;">{process.burstTime}</td>
							<td class="px-4 py-2 text-center text-white" style="border: 1px solid #4c566a;">{process.arrivalTime}</td>
							<td class="px-4 py-2 text-center text-white" style="border: 1px solid #4c566a;">{process.completionTime}</td>
							<td class="px-4 py-2 text-center text-white" style="border: 1px solid #4c566a;">{process.turnaroundTime}</td>
							<td class="px-4 py-2 text-center text-white" style="border: 1px solid #4c566a;">{process.waitingTime}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	</div>

	<!-- Average Times -->
	<div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
		<div class="rounded-lg p-4" style="background-color: #a3be8c; border: 1px solid #8fbcbb;">
			<h4 class="font-semibold mb-2" style="color: #2e3440;">Average Turnaround Time</h4>
			<p class="text-2xl font-bold" style="color: #2e3440;">{results.avgTurnaroundTime}</p>
		</div>
		<div class="rounded-lg p-4" style="background-color: #88c0d0; border: 1px solid #81a1c1;">
			<h4 class="font-semibold mb-2" style="color: #2e3440;">Average Waiting Time</h4>
			<p class="text-2xl font-bold" style="color: #2e3440;">{results.avgWaitingTime}</p>
		</div>
	</div>

	<!-- Formulas Section -->
	<div class="rounded-lg p-4" style="background-color: #434c5e; border: 1px solid #4c566a;">
		<h3 class="text-lg font-semibold mb-4 text-white">Formulas</h3>
		<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
			<div>
				<h4 class="text-md font-semibold mb-2 text-white">SRT Specific:</h4>
				<div class="space-y-2">
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">Remaining Time = Burst Time - Executed Time</div>
					</div>
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">Select shortest remaining time (preemptive)</div>
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
