<script>
	// @ts-nocheck
	export let processes = [];

	let timeQuantum = 2;
	let results = { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

	$: if (processes.length > 0) {
		calculate();
	}

	function calculate() {
		results = calculateMLQ(processes, timeQuantum);
	}

	function calculateMLQ(processes, quantum) {
		if (processes.length === 0) return { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

		const processResults = [];
		const ganttChart = [];
		let currentTime = 0;

		// Use priority values directly as queue levels (lower priority = higher precedence)
		// Default priority = 0 if not specified
		let processesWithQueues = processes.map((p) => ({
			...p,
			queueLevel: p.priority !== undefined ? p.priority : 0,
			remainingTime: p.burstTime,
		}));

		processesWithQueues.sort((a, b) => a.arrivalTime - b.arrivalTime);

		let remainingProcesses = [...processesWithQueues];
		let completedCount = 0;

		while (completedCount < processes.length) {
			// Get processes that have arrived by current time
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

			// Select process from highest priority queue (lowest queue number)
			const selectedProcess = availableProcesses.reduce((highest, current) => {
				if (current.queueLevel < highest.queueLevel) {
					return current;
				} else if (current.queueLevel === highest.queueLevel) {
					// Within same queue, use FCFS
					return current.arrivalTime < highest.arrivalTime ? current : highest;
				}
				return highest;
			});

			// Execute the process with time quantum (preemptive within queue)
			const startTime = currentTime;
			const executionTime = Math.min(selectedProcess.remainingTime, quantum);
			const endTime = currentTime + executionTime;

			ganttChart.push({
				pid: selectedProcess.pid,
				start: startTime,
				end: endTime,
				duration: executionTime,
				queueLevel: selectedProcess.queueLevel,
			});

			selectedProcess.remainingTime -= executionTime;
			currentTime = endTime;

			// Check if process is completed
			if (selectedProcess.remainingTime === 0) {
				const completionTime = currentTime;
				const turnaroundTime = completionTime - selectedProcess.arrivalTime;
				const waitingTime = turnaroundTime - selectedProcess.burstTime;

				processResults.push({
					...selectedProcess,
					completionTime,
					turnaroundTime,
					waitingTime,
				});

				completedCount++;
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

	function getQueueColor(queueLevel) {
		const colors = ["#5e81ac", "#88c0d0", "#8fbcbb"]; // System, Interactive, Batch
		return colors[queueLevel] || "#4c566a";
	}
</script>

<div class="rounded-lg shadow-lg p-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
	<h2 class="text-2xl font-semibold mb-6 text-white">Multiple Level Queue (MLQ) Scheduling</h2>

	<!-- Time Quantum Input -->
	<div class="mb-6">
		<label for="quantum" class="block text-sm font-medium mb-2 text-white">Time Quantum</label>
		<div class="flex items-center gap-3">
			<input id="quantum" type="number" bind:value={timeQuantum} min="1" class="w-32 p-2 rounded-md focus:outline-none focus:ring-2 text-white" style="background-color: #434c5e; border: 1px solid #4c566a; focus:ring-color: #d08770;" />
			<button on:click={calculate} class="text-white px-4 py-2 rounded-md hover:opacity-90 focus:outline-none focus:ring-2 transition-all duration-200 font-medium" style="background-color: #d08770; focus:ring-color: #bf616a;"> Go </button>
		</div>
	</div>

	<!-- Gantt Chart -->
	<div class="mb-8">
		<h3 class="text-lg font-semibold mb-4 text-white">Gantt Chart</h3>
		<div class="rounded-md overflow-hidden" style="background-color: #434c5e;">
			<div class="flex">
				{#each results.ganttChart as segment}
					<div class="flex flex-col items-center justify-center text-white font-medium text-sm border-r last:border-r-0" style="width: {Math.max(segment.duration * 60, 100)}px; height: 80px; background-color: {segment.pid !== 'Idle' ? getQueueColor(segment.queueLevel) : '#4c566a'}; border-color: #2e3440;" title="Process {segment.pid}: {segment.start} - {segment.end} (Duration: {segment.duration}){segment.queueType ? `, Queue: ${segment.queueType}` : ''}">
						<span>P{segment.pid}</span>
						{#if segment.queueType}
							<span class="text-xs opacity-80">Q{segment.queueLevel}</span>
						{/if}
					</div>
				{/each}
			</div>
			<div class="flex relative" style="height: 30px;">
				{#each results.ganttChart as segment, index}
					<div class="relative" style="width: {Math.max(segment.duration * 60, 100)}px;">
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
		<div class="rounded-lg p-4" style="background-color: #5e81ac; border: 1px solid #4c566a;">
			<h4 class="font-semibold mb-2" style="color: #2e3440;">Average Turnaround Time</h4>
			<p class="text-2xl font-bold" style="color: #2e3440;">{results.avgTurnaroundTime}</p>
		</div>
		<div class="rounded-lg p-4" style="background-color: #88c0d0; border: 1px solid #4c566a;">
			<h4 class="font-semibold mb-2" style="color: #2e3440;">Average Waiting Time</h4>
			<p class="text-2xl font-bold" style="color: #2e3440;">{results.avgWaitingTime}</p>
		</div>
	</div>

	<!-- Formulas Section -->
	<div class="rounded-lg p-4" style="background-color: #434c5e; border: 1px solid #4c566a;">
		<h3 class="text-lg font-semibold mb-4 text-white">Formulas</h3>
		<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
			<div>
				<h4 class="text-md font-semibold mb-2 text-white">MLQ Specific:</h4>
				<div class="space-y-2">
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">Queue Assignment = Based on priority/PID</div>
					</div>
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">Higher priority queue executes first</div>
					</div>
				</div>
			</div>
			<div>
				<h4 class="text-md font-semibold mb-2 text-white">General Scheduling:</h4>
				<div class="space-y-2">
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-green-400 text-sm">Completion Time = Start Time + Burst Time</div>
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
