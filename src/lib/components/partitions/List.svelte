<script>
	import { prettySize } from '$lib/essentials.js';

	let {
		selectedDisk = $bindable(),
		selectedPartition = $bindable(),
		modifiedPartition = $bindable(),
		originalPartition = $bindable(),
		showEdit = $bindable(),
		newPartition = $bindable(),
		newPartitionIndex = $bindable(),
		tempModifiedPartition = $bindable(),
		diskSize = $bindable(),
		diskPath = $bindable(),
		storage = $bindable(),
		maxHeight = '200'
	} = $props();

	const changeSelectedPartition = async (selected = selectedPartition) => {
		selectedPartition = selected;
		showEdit = false;
		newPartition = false;
	};

	const isUnallocated = (partition) => {
		return !partition?.path && !partition?.filesystem;
	};

	const showCreateDetail = () => {
		newPartition = true;
		showEdit = true;
	};

	const removePartition = () => {
		if (
			selectedPartition === null ||
			!modifiedPartition ||
			selectedPartition >= modifiedPartition.length
		)
			return;

		if (modifiedPartition[selectedPartition].path?.includes('#')) {

			// Update the numbering of remaining new partitions
			modifiedPartition = modifiedPartition.map((partition, i) => {
                if (selectedPartition === i) {
					return {
						...partition,
						path: null,
						filesystem: null,
						format: false,
						mountpoint: null,
						label: null,
						flags: []
					};
                }
				if (partition.path && partition.path.includes('#')) {
					const num = parseInt(partition.path.replace('#', ''));
					if (num > 1) {
						return { ...partition, path: `#${num - 1}` };
					}
				}
				return partition;
			});
		} else {
			modifiedPartition = modifiedPartition.map((partition, i) => {
				if (i === selectedPartition) {
					return {
						...partition,
						path: null,
						filesystem: null,
						format: false,
						mountpoint: null,
						label: null,
						flags: []
					};
				}
				return partition;
			});
		}

		tempModifiedPartition = modifiedPartition;
		showEdit = false;
		newPartition = false;

        if (selectedPartition === tempModifiedPartition.length) selectedPartition -= 1;
	};

	const editPartition = () => {
		if (
			selectedPartition === null ||
			!modifiedPartition ||
			selectedPartition >= modifiedPartition.length
		)
			return;
		showEdit = true;
		newPartition = false;
	};
</script>

<div
	class="flex flex-col justify-between w-[1050px] rounded-[13px] border-[1.3px] border-[#3C6350] bg-[#101010] p-4"
>
	<div class="w-full overflow-y-auto max-h-[{maxHeight}px]">
		<table class="w-full">
			<thead class="text-[#FFFEFB] font-['Poppins'] text-[14px]">
				<tr class="border-b border-[#3C6350]">
					<th class="p-3 text-left">Device</th>
					<th class="p-3 text-left">Size</th>
					<th class="p-3 text-left">Format</th>
					<th class="p-3 text-left">Type</th>
					<th class="p-3 text-left">MountPoint</th>
				</tr>
			</thead>
			<tbody class="text-[#FFFEFB] font-['Poppins'] text-[14px]">
				{#key selectedDisk}
					{#if modifiedPartition}
						{#each modifiedPartition as p, num}
							<tr
								on:click={() => changeSelectedPartition(num)}
								class="border-b border-[#3C6350] {num === selectedPartition
									? 'bg-[rgba(38,167,104,0.2)]'
									: 'bg-[#101010]'}"
								style="cursor: default;"
							>
								<td class="p-3"
									>{p.path
										? p.path.includes('#')
											? `New Partition ${p.path}`
											: p.path
										: 'Unallocated'}</td
								>
								<!-- <td class="p-3">{prettySize(p.size)}</td> -->
								<td class="p-3">{p?.size ? prettySize(p.size) : ''}</td>
								<td class="p-3">{p.format ? 'Yes' : 'No'}</td>
								<td class="p-3"
									>{p.filesystem ? p.filesystem : p.path ? 'Unknown' : 'Unallocated'}</td
								>
								<td class="p-3">{p.mountpoint ? p.mountpoint : ''}</td>
							</tr>
						{/each}
					{/if}
				{/key}
			</tbody>
		</table>
	</div>

	<div class="flex justify-between mt-4">
		<button
			class="flex h-8 px-[9px] items-center justify-center gap-[10px] rounded-[4px] border-[0.3px] border-[#3C6350] bg-[#101010] text-white font-['Poppins'] text-[14px] transition-all duration-200 hover:shadow-[0_0_9px_#00B85E] active:shadow-[0_0_9px_#00B85E] disabled:opacity-50 disabled:hover:shadow-none"
			on:click={showCreateDetail}
            disabled={modifiedPartition[selectedPartition].path !== null}
		>
			+ Add
		</button>
		<div class="flex gap-3">
			<button
				class="flex h-8 px-[9px] items-center justify-center gap-[10px] rounded-[4px] border-[0.3px] border-[#3C6350] bg-[#101010] text-white font-['Poppins'] text-[14px] transition-all duration-200 hover:shadow-[0_0_9px_#00B85E] active:shadow-[0_0_9px_#00B85E] disabled:opacity-50 disabled:hover:shadow-none"
				on:click={editPartition}
				disabled={selectedPartition === null ||
					isUnallocated(modifiedPartition?.[selectedPartition])}
			>
				Edit
			</button>
			<button
				class="flex h-8 px-[9px] items-center justify-center gap-[10px] rounded-[4px] border-[0.3px] border-[#633C3C] bg-[#101010] text-white font-['Poppins'] text-[14px] transition-all duration-200 hover:shadow-[0_0_9px_#FF453A] active:shadow-[0_0_9px_#FF453A] disabled:opacity-50 disabled:hover:shadow-none"
				on:click={removePartition}
				disabled={selectedPartition === null || !modifiedPartition?.[selectedPartition] || !modifiedPartition?.[selectedPartition].path}
			>
				Delete
			</button>
		</div>
	</div>
</div>
