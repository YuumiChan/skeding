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

<div class="rounded-lg shadow-lg p-6" style="background-color: #3b4252; border: 1px solid #434c5e;">
	<h2 class="text-2xl font-semibold mb-6 text-white">First Come First Serve (FCFS) Scheduling</h2>

	<!-- Gantt Chart -->
	<div class="mb-8">
		<h3 class="text-lg font-semibold mb-4 text-white">Gantt Chart</h3>
		<div class="rounded-md overflow-hidden" style="background-color: #434c5e;">
			<div class="flex">
				{#each results.ganttChart as segment}
					<div
						class="flex items-center justify-center text-white font-medium text-sm border-r last:border-r-0"
						style="width: {Math.max(segment.duration * 60, 80)}px; height: 60px; background-color: {segment.pid !== 'Idle'
							? '#88c0d0'
							: '#4c566a'}; border-color: #2e3440;"
						title="Process {segment.pid}: {segment.start} - {segment.end} (Duration: {segment.duration})"
					>
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
	<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
		<div class="rounded-lg p-4" style="background-color: #a3be8c; border: 1px solid #8fbcbb;">
			<h4 class="font-semibold mb-2" style="color: #2e3440;">Average Turnaround Time</h4>
			<p class="text-2xl font-bold" style="color: #2e3440;">{results.avgTurnaroundTime}</p>
		</div>
		<div class="rounded-lg p-4" style="background-color: #88c0d0; border: 1px solid #81a1c1;">
			<h4 class="font-semibold mb-2" style="color: #2e3440;">Average Waiting Time</h4>
			<p class="text-2xl font-bold" style="color: #2e3440;">{results.avgWaitingTime}</p>
		</div>
	</div>
</div>
