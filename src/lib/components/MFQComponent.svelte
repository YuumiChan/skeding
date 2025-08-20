<script>
	// @ts-nocheck
	export let processes = [];

	let quantum1 = 2;
	let quantum2 = 4;
	let queueTypes = ["RR(q=2)", "RR(q=4)", "FCFS"];
	let timeQuantums = [2, 4, Infinity];
	let results = { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

	// Update queue types and quantums when input changes
	$: {
		queueTypes = [`RR(q=${quantum1})`, `RR(q=${quantum2})`, "FCFS"];
		timeQuantums = [quantum1, quantum2, Infinity];
	}

	$: if (processes.length > 0) {
		calculate();
	}

	function calculate() {
		results = calculateMFQ(processes);
	}

	function calculateMFQ(processes) {
		if (processes.length === 0) return { processResults: [], ganttChart: [], avgTurnaroundTime: 0, avgWaitingTime: 0 };

		const processResults = [];
		const ganttChart = [];
		let currentTime = 0;

		// Initialize processes with queue level 0 and remaining time
		let processesWithQueues = processes.map((p) => ({
			...p,
			queueLevel: 0, // Start all processes in highest priority queue
			remainingTime: p.burstTime,
			completionTime: null,
		}));

		// Create separate queues for each level
		let queues = [[], [], []]; // 3 queue levels
		let remainingProcesses = [...processesWithQueues];
		let completedCount = 0;

		// Sort processes by arrival time
		remainingProcesses.sort((a, b) => a.arrivalTime - b.arrivalTime);
		let processIndex = 0;

		while (completedCount < processes.length) {
			// Add newly arrived processes to queue 0
			while (processIndex < remainingProcesses.length && remainingProcesses[processIndex].arrivalTime <= currentTime) {
				if (remainingProcesses[processIndex].remainingTime > 0) {
					queues[0].push(remainingProcesses[processIndex]);
				}
				processIndex++;
			}

			// Find the highest priority non-empty queue
			let currentQueueLevel = -1;
			for (let i = 0; i < queues.length; i++) {
				if (queues[i].length > 0) {
					currentQueueLevel = i;
					break;
				}
			}

			if (currentQueueLevel === -1) {
				// No process in any queue, advance time to next arrival
				if (processIndex < remainingProcesses.length) {
					const nextArrival = remainingProcesses[processIndex].arrivalTime;
					if (currentTime < nextArrival) {
						ganttChart.push({
							pid: "Idle",
							start: currentTime,
							end: nextArrival,
							duration: nextArrival - currentTime,
						});
						currentTime = nextArrival;
					}
				}
				continue;
			}

			// Get process from the highest priority queue
			const currentProcess = queues[currentQueueLevel].shift();
			const quantum = timeQuantums[currentQueueLevel];
			const executionTime = Math.min(currentProcess.remainingTime, quantum);

			// Execute process
			ganttChart.push({
				pid: currentProcess.pid,
				start: currentTime,
				end: currentTime + executionTime,
				duration: executionTime,
				queueLevel: currentQueueLevel,
				queueType: queueTypes[currentQueueLevel],
			});

			currentProcess.remainingTime -= executionTime;
			currentTime += executionTime;

			// Check if process completed
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
				// Process not completed, demote to next queue level
				const nextQueueLevel = Math.min(currentQueueLevel + 1, queues.length - 1);
				currentProcess.queueLevel = nextQueueLevel;
				queues[nextQueueLevel].push(currentProcess);
			}

			// Add newly arrived processes during execution
			while (processIndex < remainingProcesses.length && remainingProcesses[processIndex].arrivalTime <= currentTime) {
				if (remainingProcesses[processIndex].remainingTime > 0) {
					queues[0].push(remainingProcesses[processIndex]);
				}
				processIndex++;
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
		const colors = ["#bf616a", "#d08770", "#ebcb8b"]; // Queue 0, 1, 2
		return colors[queueLevel] || "#4c566a";
	}
</script>

<div class="rounded-lg shadow-lg p-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
	<h2 class="text-2xl font-semibold mb-6 text-white">Multilevel Feedback Queue (MFQ) Scheduling</h2>

	<!-- Time Quantum Configuration -->
	<div class="mb-6 p-4 rounded-lg" style="background-color: #434c5e; border: 1px solid #4c566a;">
		<h3 class="text-lg font-semibold mb-3 text-white">Configure Time Quantums</h3>
		<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
			<div>
				<label for="quantum1" class="block text-sm font-medium mb-2 text-white">Queue 0 Time Quantum</label>
				<div class="flex items-center gap-3">
					<input id="quantum1" type="number" bind:value={quantum1} min="1" class="w-24 p-2 rounded-md focus:outline-none focus:ring-2 text-white" style="background-color: #2e3440; border: 1px solid #4c566a;" />
					<span class="text-sm text-gray-300">time units</span>
				</div>
			</div>
			<div>
				<label for="quantum2" class="block text-sm font-medium mb-2 text-white">Queue 1 Time Quantum</label>
				<div class="flex items-center gap-3">
					<input id="quantum2" type="number" bind:value={quantum2} min="1" class="w-24 p-2 rounded-md focus:outline-none focus:ring-2 text-white" style="background-color: #2e3440; border: 1px solid #4c566a;" />
					<span class="text-sm text-gray-300">time units</span>
				</div>
			</div>
		</div>
		<p class="text-xs text-gray-400 mt-2">Queue 2 uses FCFS (no time limit)</p>
	</div>

	<!-- Queue Information -->
	<div class="mb-6 p-4 rounded-lg" style="background-color: #434c5e; border: 1px solid #4c566a;">
		<h3 class="text-lg font-semibold mb-3 text-white">Queue Levels & Time Quantums</h3>
		<div class="grid grid-cols-1 md:grid-cols-3 gap-3">
			{#each queueTypes as queueType, index}
				<div class="p-3 rounded-lg text-center" style="background-color: {getQueueColor(index)};">
					<div class="font-semibold text-white">Queue {index}</div>
					<div class="text-sm text-gray-200">{queueType}</div>
					<div class="text-xs text-gray-300 mt-1">
						{index === 0 ? "Highest Priority" : index === 1 ? "Medium Priority" : "Lowest Priority"}
					</div>
					<div class="text-xs text-gray-300">
						{index === 2 ? "No time limit" : `Quantum: ${timeQuantums[index]}`}
					</div>
				</div>
			{/each}
		</div>
		<div class="mt-3 text-sm text-gray-300">
			• New processes start in Queue 0<br />
			• Process moves to next queue if it doesn't complete within time quantum<br />
			• Higher priority queues preempt lower priority ones
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
						<th class="px-4 py-3 text-left text-white font-semibold" style="border: 1px solid #4c566a;">Priority</th>
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
							<td class="px-4 py-2 text-center text-white" style="border: 1px solid #4c566a;">{process.priority || "N/A"}</td>
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
		<div class="rounded-lg p-4" style="background-color: #bf616a; border: 1px solid #a3515a;">
			<h4 class="font-semibold mb-2" style="color: #2e3440;">Average Turnaround Time</h4>
			<p class="text-2xl font-bold" style="color: #2e3440;">{results.avgTurnaroundTime}</p>
		</div>
		<div class="rounded-lg p-4" style="background-color: #d08770; border: 1px solid #b8735e;">
			<h4 class="font-semibold mb-2" style="color: #2e3440;">Average Waiting Time</h4>
			<p class="text-2xl font-bold" style="color: #2e3440;">{results.avgWaitingTime}</p>
		</div>
	</div>

	<!-- Formulas Section -->
	<div class="rounded-lg p-4" style="background-color: #434c5e; border: 1px solid #4c566a;">
		<h3 class="text-lg font-semibold mb-4 text-white">Formulas</h3>
		<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
			<div>
				<h4 class="text-md font-semibold mb-2 text-white">MFQ Specific:</h4>
				<div class="space-y-2">
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">New processes → Queue 0 (highest priority)</div>
					</div>
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">Incomplete process → Next lower queue</div>
					</div>
					<div class="p-2 rounded" style="background-color: #2e3440; font-family: monospace;">
						<div class="text-yellow-400 text-sm">Higher queues preempt lower queues</div>
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
