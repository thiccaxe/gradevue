<script lang="ts">
	import { browser } from '$app/environment';
	import { loadDocumentsList } from '$lib/cache';
	import DateBadge from '$lib/components/DateBadge.svelte';
	import LoadingBanner from '$lib/components/LoadingBanner.svelte';
	import { documentsList, documentsListLoaded } from '$lib/stores';
	import { Badge, Button, Card, TabItem, Tabs } from 'flowbite-svelte';
	import { DownloadSolid } from 'flowbite-svelte-icons';
	import { studentAccount } from '$lib/stores';

	if (!$documentsList && browser) loadDocumentsList();

	function getDocumentColor(documentType: string) {
		switch (documentType) {
			case 'Report Card':
				return 'yellow';
			case 'Transcript':
				return 'red';
			case 'MAP Growth Family Report':
				return 'blue';
			default:
				return 'primary';
		}
	}

	$: documents = $documentsList?.StudentDocumentDatas?.StudentDocumentData ?? [];

	const sortPriority = ['Transcript', 'Report Card'];

	$: documentCategories = new Set(
		documents
			.map((document) => document._DocumentType)
			.toSorted((a, b) => {
				const aPriority = sortPriority.indexOf(a);
				const bPriority = sortPriority.indexOf(b);
				if (aPriority == -1 && bPriority == -1) return a.localeCompare(b);
				if (aPriority == -1) return 1;
				if (bPriority == -1) return -1;
				return aPriority - bPriority;
			})
	);
	const downloadDocument = async (documentGUID: string, documentTitle: string) => {

		const documentData = await $studentAccount?.reportCard(documentGUID);
		if (!documentData) {
			console.error("invalid report card data");
			return;
		}
		const dummyResp = await fetch(`data:application/pdf;base64,${documentData?.Base64Code}`);
		if (!dummyResp) {
			console.error("error in converting base64 to blob");
			return;
		}
		const blob = await dummyResp.blob();
		const url = URL.createObjectURL(blob);
		const downloadLink = document.createElement("a");
		downloadLink.title = documentTitle;
		downloadLink.href = url;
		downloadLink.target = "_blank";
		downloadLink.download = documentTitle;
		document.body.appendChild(downloadLink);
		downloadLink.click();


		
	}
</script>

<LoadingBanner show={!$documentsListLoaded} loadingMsg="Loading documents..." />

{#if $documentsList}
	<Tabs class="m-4 mb-0" contentClass="p-4">
		<TabItem title="All" open>
			<ol class="space-y-4">
				{#each documents as document}
					<li class="flex flex-row gap-x-4">
						<Card
							href="/documents/{document._DocumentGU}"
							class="dark:text-white max-w-none flex flex-row items-center gap-2 flex-wrap"
						>
							<h2 class="text-md">{document._DocumentComment}</h2>
							<DateBadge date={new Date(document._DocumentDate)} />
							<Badge color={getDocumentColor(document._DocumentType)}>
								{document._DocumentType}
							</Badge>
						</Card>
						<Button size="sm" color="alternative" outline on:click={async () => await downloadDocument(document._DocumentGU, document._DocumentComment)}><DownloadSolid class="w-4 md:w-6 lg:w-8" title={{title: "download"}}/></Button>
					</li>
				{/each}
			</ol>
		</TabItem>
		{#each documentCategories as category}
			<TabItem title={category}>
				<ol class="space-y-4">
					{#each documents.filter((document) => document._DocumentType == category) as document}
						<li class="flex flex-row gap-x-4">
							<Card
								href="/documents/{document._DocumentGU}"
								class="dark:text-white max-w-none flex flex-row items-center gap-2 flex-wrap"
							>
								<h2 class="text-md">{document._DocumentComment}</h2>
								<DateBadge date={new Date(document._DocumentDate)} />
							</Card>
							<Button color="alternative" outline on:click={async () => await downloadDocument(document._DocumentGU, document._DocumentComment)}><DownloadSolid title={{title: "download"}}/></Button>
						</li>
					{/each}
				</ol>
			</TabItem>
		{/each}
	</Tabs>
{/if}
