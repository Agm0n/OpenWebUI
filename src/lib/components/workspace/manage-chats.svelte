<script>
    import { deleteTagById, addTagById, getAllChats, getTagsById, updateChatById } from "$lib/apis/chats";
	import i18n from "$lib/i18n";
	import { onMount } from "svelte";
	import { writable } from "svelte/store";
	import Pencil from "../icons/Pencil.svelte";
	import Modal from "../common/Modal.svelte";
    import { goto, invalidateAll } from '$app/navigation';
	import { toast } from "svelte-sonner";
	import Tooltip from "../common/Tooltip.svelte";
	import { user } from "$lib/stores";


	let chats = [];
    
    let show = false;
    let currName = "";
    let currChat = null;
    let currTags = [];
    let newTagId = "";
    let newTagDisplay = "";

    let addedTags = [];
    let removedTags = [];

	let sortColumn = writable('title');
	let sortDirection = writable('asc');
    

	onMount(async () => {
		let fetchedChats = await getAllChats(localStorage.token);
        chats = await Promise.all(
            fetchedChats.map(async (chat) => {
                const tags = await getTagsById(localStorage.token, chat.id);
                return { ...chat, tags };
            })
        );
	});

	function sortBy(column) {
		sortColumn.update(c => {
			if (c === column) {
				sortDirection.update(d => d === 'asc' ? 'desc' : 'asc');
			} else {
				sortDirection.set('asc');
			}
			return column;
		});
	}

	$: sortedChats = [...chats].sort((a, b) => {
		const column = $sortColumn;
		const dir = $sortDirection === 'asc' ? 1 : -1;

		let aValue = a[column];
		let bValue = b[column];

		if (column === 'updated_at' || column === 'created_at') {
			aValue = new Date(aValue).getTime();
			bValue = new Date(bValue).getTime();
		}

        if (column === 'tags') {
            aValue = aValue.length;
            bValue = bValue.length;
        }

		if (aValue < bValue) return -1 * dir;
		if (aValue > bValue) return 1 * dir;
		return 0;
	});

</script>

<svelte:head>
	<title>{$i18n.t('Manage Chats')}</title>
</svelte:head>


<Modal 
    bind:show
    title={$i18n.t('Edit chat')}
    showClose={true}
>
    <div class="p-6">
        <div class="mb-4">
            <p class="text-sm text-gray-600 dark:text-gray-300 mb-2 text-center">
                {$i18n.t('Enter a new name for the chat')}
            </p>
        </div>
        
        <div class="mb-4">
            <input type="text"
                bind:value={currName}
                placeholder={$i18n.t('Enter new name here...')}
                class="w-full text-sm outline-hidden resize-vertical border-2 border-gray-300 dark:border-gray-800 rounded-lg p-3"
            />
        </div>

        <div class="mb-4">
        <p class="text-sm text-gray-600 dark:text-gray-300 mb-2 text-center">
            {$i18n.t('Edit tags')}
        </p>

        <div class="flex flex-wrap justify-center gap-2 mb-2">
            {#each currChat?.tags || [] as tag, idx}
                <div class="flex items-center bg-gray-200 dark:bg-gray-700 text-gray-700 dark:text-gray-300 rounded-3xl px-3 py-1 text-xs font-medium">
                    {tag.name}
                    <button
                        class="ml-2 text-gray-500 hover:text-red-500"
                        on:click={() => {
                            currChat.tags = currChat.tags.filter((_, i) => i !== idx);
                            removedTags.push(tag);
                        }}
                    >
                        ✕
                    </button>
                </div>
            {/each}
        </div>

        <div class="flex justify-center gap-2 mb-2">
            <!-- <input
                type="text"
                bind:value={newTagId}
                placeholder={$i18n.t('ID Name')}
                class="text-sm border border-gray-300 dark:border-gray-800 rounded-lg p-2 w-1/3"
            /> -->
            <input
                type="text"
                bind:value={newTagDisplay}
                placeholder={$i18n.t('Enter tag name')}
                class="text-sm border border-gray-300 dark:border-gray-800 rounded-lg p-2 w-1/3"
            />
            <button
                class="px-3 py-2 text-sm bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200 rounded-lg hover:bg-gray-300 dark:hover:bg-gray-600 transition"
                on:click={() => {
                    if (newTagDisplay.trim()) {
                        newTagId = newTagDisplay;
                        
                        currChat.tags = [
                            ...(currChat.tags ?? []),
                            { name: newTagDisplay.trim(), id: newTagId.trim(), meta: undefined, user_id: $user?.id }
                        ];
                        addedTags.push({ name: newTagDisplay.trim(), id: newTagId.trim(), meta: undefined, user_id: $user?.id });//I left it the entire tag object even though only the name is needed because I think it is wierd to update the tags only by name (but I am not going to change it now)
                        newTagDisplay = "";
                        newTagId = "";
                    }
                }}
            >
                {$i18n.t('Add')}
            </button>
        </div>

        <div class="flex justify-center gap-2">
            <button
                class="px-4 py-2 text-sm text-gray-600 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-800 rounded-lg transition-colors"
                on:click={() => {
                    show = false;
                }}
            >
                {$i18n.t('Cancel')}
            </button>
            <button
                class="px-4 py-2 text-sm text-white bg-black hover:bg-gray-900 dark:bg-white dark:text-black dark:hover:bg-gray-100 rounded-lg transition-colors"
                on:click={() => {
                    if(!currChat) return;
                    updateChatById(localStorage.token, currChat.id, { ...currChat, title: currName })
                    .then(() => {
                        show = false;

                        return Promise.all(
                            removedTags.map(tag => deleteTagById(localStorage.token, currChat.id, tag.id))
                        );
                    })
                    .then(() => {
                        removedTags = [];
                        return Promise.all(
                            addedTags.map(tag => addTagById(localStorage.token, currChat.id, tag.name))
                        );
                    })
                    .then(() => {
                        addedTags = [];
                        toast.success($i18n.t('Chat updated successfully'));

                        invalidateAll();

                        // Reloading the page (temporary solution)
                        location.reload();
                    })
                    .catch(err => {
                        console.error(err);
                        toast.error($i18n.t('Failed to update chat'));
                    });
                }}
            >
                {$i18n.t('Save')}
            </button>
        </div>
    </div>
</Modal>


<div class="flex flex-col gap-1 px-1 mt-1.5 mb-3">
	<div class="flex justify-between items-center">
		<div class="flex items-center md:self-center text-xl font-medium px-0.5 gap-2 shrink-0">
			<div>
				{$i18n.t('Manage Chats')}
			</div>
		</div>
	</div>
</div>

<div class="py-2 bg-white dark:bg-gray-900 rounded-3xl border border-gray-100 dark:border-gray-850">
	<div class="flex w-full flex-col gap-4 py-0.5 px-3.5 pb-2">
		{#if chats.length === 0}
			<p>{$i18n.t('No chats available.')}</p>
		{:else}
			<div class="flex justify-between text-gray-500 dark:text-gray-300 font-medium text-sm px-3 py-1">
				<div class="flex-1 cursor-pointer" on:click={() => sortBy('title')}>
					{$i18n.t('Name')}
					{#if $sortColumn === 'title'}
						{#if $sortDirection === 'asc'} ▲ {:else} ▼ {/if}
					{/if}
				</div>
                <div class="flex-1 cursor-pointer" on:click={() => sortBy('tags')}>
					{$i18n.t('Tags')}
					{#if $sortColumn === 'tags'}
						{#if $sortDirection === 'asc'} ▲ {:else} ▼ {/if}
					{/if}
				</div>
				<div class="flex-1 cursor-pointer" on:click={() => sortBy('updated_at')}>
					{$i18n.t('Last Update')}
					{#if $sortColumn === 'updated_at'}
						{#if $sortDirection === 'asc'} ▲ {:else} ▼ {/if}
					{/if}
				</div>
				<div class="flex-1 cursor-pointer" on:click={() => sortBy('created_at')}>
					{$i18n.t('Creation Date')}
					{#if $sortColumn === 'created_at'}
						{#if $sortDirection === 'asc'} ▲ {:else} ▼ {/if}
					{/if}
				</div>
			</div>

			{#each sortedChats as chat, i}
				<div class="py-2 bg-white dark:bg-gray-900 rounded-3xl border border-gray-100 dark:border-gray-850 flex flex-row items-center hover:bg-gray-500 dark:hover:bg-gray-800 cursor-pointer transition-pointers"
                    on:click={() => {
                        toast($i18n.t('Chat selected') + ": " + chat.title);
                        goto(`/c/${chat.id}`);
                    }}>
					<div class="flex-1 text-sm font-medium text-gray-900 dark:text-white px-3">{chat.title}</div>
                    <div class="flex-1 text-sm font-medium text-gray-900 dark:text-white px-3 gap-1">
                        {#each chat.tags as tag}
                            <div 
                             class="inline-block px-2 py-0.5 mx-0.5 my-1 text-xs font-medium bg-gray-200 text-gray-700 rounded-3xl dark:bg-gray-700 dark:text-gray-300 cursor-text"
                              on:click={(e) => {
                                e.stopPropagation();
                             }}>
                                {tag.name}
                            </div>
                        {/each}
                    </div>
					<div class="flex-1 text-sm text-gray-500 dark:text-gray-300">{new Date(chat.updated_at * 1000).toLocaleString()}</div>
					<div class="flex-1 text-sm text-gray-500 dark:text-gray-300">{new Date(chat.created_at * 1000).toLocaleString()}</div>
                    <Tooltip content={$i18n.t('Edit chat')} placement="top">
                        <button
                            on:click={(e) => {
                                e.stopPropagation();
                                currChat = chat;
                                currName = chat.title;
                                currTags = chat.tags ?? [];
                                show = true;
                            }}
                            class="p-2 m-1 bg-gray-100 dark:bg-gray-900 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700 transition">
                            <Pencil />
                        </button>
                    </Tooltip>
				</div>
			{/each}
		{/if}
	</div>
</div>
