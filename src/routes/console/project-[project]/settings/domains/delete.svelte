<script lang="ts">
    import { Modal } from '$lib/components';
    import { Button } from '$lib/elements/forms';
    import { project } from '../../store';
    import { sdkForConsole } from '$lib/stores/sdk';
    import { addNotification } from '$lib/stores/notifications';
    import { invalidate } from '$app/navigation';
    import { Dependencies } from '$lib/constants';
    import type { Models } from '@aw-labs/appwrite-console';
    import { trackEvent } from '$lib/actions/analytics';

    export let showDelete = false;
    export let selectedDomain: Models.Domain;

    const deleteDomain = async () => {
        try {
            await sdkForConsole.projects.deleteDomain($project.$id, selectedDomain.$id);
            invalidate(Dependencies.DOMAINS);
            showDelete = false;
            addNotification({
                type: 'success',
                message: `${selectedDomain.domain} has been deleted`
            });
            trackEvent('submit_domain_delete');
        } catch (error) {
            addNotification({
                type: 'error',
                message: error.message
            });
        }
    };
</script>

<Modal bind:show={showDelete} on:submit={deleteDomain} warning>
    <svelte:fragment slot="header">Delete Domain</svelte:fragment>
    {#if selectedDomain}
        <p>
            Are you sure you want to delete <b>{selectedDomain.domain}</b> from '{$project.name}'?
        </p>
    {/if}
    <svelte:fragment slot="footer">
        <Button text on:click={() => (showDelete = false)}>Cancel</Button>
        <Button secondary submit>Delete</Button>
    </svelte:fragment>
</Modal>
