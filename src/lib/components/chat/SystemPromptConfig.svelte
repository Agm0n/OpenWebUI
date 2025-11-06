<script lang="ts">
    import { getContext, createEventDispatcher } from 'svelte';
    import { toast } from 'svelte-sonner';
    import Settings from '$lib/components/icons/Settings.svelte';
    import Tooltip from '$lib/components/common/Tooltip.svelte';
    import Modal from '$lib/components/common/Modal.svelte';

    const i18n = getContext('i18n');
    
    const dispatch = createEventDispatcher();

    let show = false;
    let systemPrompt = '';

</script>

<Modal 
    bind:show
    title={$i18n.t('Configure System Prompt')}
    showClose={true}
>
    <div class="p-6">
        <div class="mb-4">
            <p class="text-sm text-gray-600 dark:text-gray-300 mb-2 text-center">
                {$i18n.t('Configure the system prompt that will be used for this conversation')}
            </p>
        </div>
        
        <div class="mb-4">
            <textarea
                bind:value={systemPrompt}
                placeholder={$i18n.t('Enter your system prompt here...')}
                class="w-full text-sm outline-hidden resize-vertical border-2 border-gray-300 dark:border-gray-800 rounded-lg p-3"
            />
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
                    dispatch('set-system-prompt', systemPrompt);

                    toast.success($i18n.t('System prompt updated successfully'));
                    show = false;
                    
                }}
            >
                {$i18n.t('Save')}
            </button>
        </div>
    </div>
</Modal>

<Tooltip content={$i18n.t('Configure system prompt')} placement="top">
    <button 
        class="bg-transparent hover:bg-gray-100 text-gray-700 dark:text-white dark:hover:bg-gray-800 rounded-full size-8 flex justify-center items-center outline-hidden focus:outline-hidden"
        type="button"
        on:click={() => {
            show = true;
        }}
    >
        <Settings />
    </button>
</Tooltip>