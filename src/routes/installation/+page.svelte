<script>
	import { invoke } from '@tauri-apps/api/core';
	import { onMount } from 'svelte';
	import { getRead } from './global.js';
	import prettyBytes from 'pretty-bytes';
	import { randomColor } from 'randomcolor';
	import TwoSide from '$lib/components/layouts/TwoSide.svelte';
	import GlowingText from '$lib/components/ui/GlowingText.svelte';
	import DiskSlider from '../../lib/components/DiskSlider.svelte';
	import Navigation from '../../lib/components/Navigation.svelte';
	import { showModal } from '$lib/stores/modalStore';

	const showSuccessModal = () => {
		showModal({
			title: 'Success!',
			content: 'Your action was completed successfully.',
			type: 'success'
		});
	};

	const showConfirmationModal = () => {
		showModal({
			title: 'Are you sure?',
			content: 'This action cannot be undone.',
			type: 'warning',
			showCancel: true,
			onConfirm: () => {
				console.log('Confirmed!');
			}
		});
	};

	const showErrorModal = () => {
		showModal({
			isOpen: false,
			type: 'error',
			title: 'Error !',
			content: 'ini error',
			confirmText: 'OK',
			cancelText: 'Cancel',
			showCancel: true,
			onConfirm: null
		});
	};

	const getStorageJSON = async () => {
		let json = await getRead();

		json = json?.disk
                ? json.disk.length
                    ? json.disk
                    : null
                : null
            ?? null;

		return json;
	};

	const getTotalStorage = async () => {
		let storage = await getStorageJSON();
		let total = 0;

        if (storage && storage.length) {
            for (let i of storage.keys()) {
                let size = storage[i].size.slice(0, -1);

                total += parseInt(size);
            }
        }

		return total;
	};

	const getColors = (disks, partIdx) => {
        if (disks[partIdx].partitions) {

            let length = disks[partIdx].partitions.length;

            let colors = [];

            for (let i = 0; i < length; i++) {
                colors.push(
                    randomColor({
                        luminosity: 'bright',
                        hue: 'random'
                    })
                );
            }
            return colors;
        }

        return "#454545";
	};

	const checkUnknown = (s) => {
		if (!s || s.length === 0) {
			return 'Unknown';
		} else {
			return s;
		}
	};

	// onMount(() => {
	// 	getStorageJSON().then((disks) => {
	// 		getColors(disks, 0);
	// 	});
	// });
</script>

{#await getRead() then json}
	<!-- <pre>
		{JSON.stringify(json, null, 2)}
	</pre> -->
	<TwoSide>
		{#snippet left()}
			<div class="w-[288px] space-y-[15px]">
				<div class="flex space-x-[14px]">
					<div class="w-[58px]">
						<img src="/logo-tealinux.svg" class="w-full" alt="logo" />
					</div>
					<h1 class="font-archivo font-[600] text-[40px] tracking-[-1.8px]">TeaLinux OS</h1>
				</div>
				<p class="font-jakarta text-sm font-[200] tracking-[-0.56px] text-center">
                    <i>"Nikmatnya sebuah racikan"</i>
				</p>
			</div>
		{/snippet}
		{#snippet right()}
			<!-- information system -->
			<div class="flex space-x-2 mb-[8px] mt-[5px]">
				<div
					class="w-1/2 bg-[#101010] border-[1.3px] border-[#3C6350] p-[15px] rounded-[14px] space-y-4"
				>
					<GlowingText size="[15]" text="Hardware" />
					<div class="space-y-4 text-[15px]">
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Hardware model</p>
							<p class="font-[200]">
								{checkUnknown(json.model.systemVersion)}-{checkUnknown(
									json.model.systemProductName
								)}
							</p>
						</div>
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Memory</p>
							<p class="font-[200]">
								{parseFloat((json.memory.capacity / 1024).toFixed(2))} GiB
							</p>
						</div>
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Processor</p>
							<p class="font-[200]">
								{checkUnknown(json.lspci.cpu)}
							</p>
						</div>
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Primary Graphic Card</p>
							<p class="font-[200]">{checkUnknown(json.lspci.vga[0])}</p>
						</div>

						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Secondary Graphic Card</p>
							<p class="font-[200]">{json.lspci.vga[1] || '-'}</p>
						</div>
						{#await getTotalStorage() then totalSize}
							{@const storage = totalSize * 512}
							{@const storageGB = storage === 0 ? '-' : prettyBytes(totalSize * 512)}
							<div class="leading-none space-y-[10px]">
								<p class="font-[500]">Disk Capacity</p>
								<p class="font-[200]">{storageGB}</p>
							</div>
						{/await}
					</div>
				</div>

				<div
					class="w-1/2 bg-[#101010] border-[1.3px] border-[#3C6350] p-[15px] rounded-[14px] space-y-5"
				>
					<GlowingText size="[15]" text="Operating System" />
					<div class="space-y-4 text-[15px]">
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Operating System</p>
							<p class="font-[200]">{json.operatingSystem.name}</p>
						</div>
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Operating System Architecture</p>
							<p class="font-[200]">{json.operatingSystem.architecture}</p>
						</div>
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Kernel</p>
							<p class="font-[200]">{json.kernel.name} {json.kernel.version}</p>
						</div>
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Boot Mode</p>
							<p class="font-[200]">{checkUnknown(json.firmware)}</p>
						</div>
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Desktop Environment</p>
							<p class="font-[200] capitalize">{json.desktopEnvironment.name}</p>
						</div>
						<div class="leading-none space-y-[10px]">
							<p class="font-[500]">Display Server</p>
							<p class="font-[200] capitalize">{json.displayServer.name}</p>
						</div>
					</div>
				</div>
			</div>
			{#await getStorageJSON()}
				Loading...
			{:then disks}
				<DiskSlider {disks} />
			{/await}
		{/snippet}
	</TwoSide>
{/await}

<Navigation
	currentStep={1}
	currentTitle="System Information"
	prevPath="/installation"
	nextPath="/installation/localization"
	nextAction={null}
/>
