<script lang="ts">
	import { browser } from '$app/environment';
	import { loadMessages } from '$lib/cache';
	import { studentAccount } from '$lib/stores';
	import DateBadge from '$lib/components/DateBadge.svelte';
	import LoadingBanner from '$lib/components/LoadingBanner.svelte';
	import { messages, messagesLoaded } from '$lib/stores';
	import type { Message } from '$lib/types/Message';
	import { Badge, Card } from 'flowbite-svelte';
	import { UserOutline } from 'flowbite-svelte-icons';

	if (!$messages && browser) loadMessages();

	async function openMessageWindow() {
		const resp = await $studentAccount?.authToken()
		console.log(resp);	
		const token = resp?._EncyToken;
		const url = new URL(`https://${$studentAccount?.domain}/PXP2_Messages.aspx`);
		const queryParams = url.searchParams;
		queryParams.append("token", token||"");
		queryParams.append("LNG", "00");
		queryParams.append("regenerateSessionId", "True");
		queryParams.append("mobile", "False");
		window.open(url, "_gradevue_messages");
		

		// get the token

	}
</script>

<LoadingBanner show={!$messagesLoaded} loadingMsg="Loading messages..." />

{#if $messages}
	<ol class="p-4 space-y-4">
		{#each $messages as message}
			<li>
				<Card title="Open this message in a new tab" on:click={openMessageWindow} class="dark:text-white max-w-none flex flex-row items-center gap-2 flex-wrap hover:cursor-pointer">
					<h2 class="text-md">{message._SubjectNoHTML}</h2>
					<Badge color="blue">
						<UserOutline size="xs" class="focus:outline-none mr-1" />
						{message._Email}
					</Badge>
					<DateBadge date={new Date(message._BeginDate)} />
					<Badge color="dark">{message._Type}</Badge>
				</Card>
			</li>
		{/each}
	</ol>
{/if}
