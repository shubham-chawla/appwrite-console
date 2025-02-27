<script lang="ts">
    import { Button, FormList, InputFile } from '$lib/elements/forms';
    import { Pill } from '$lib/elements';
    import { Modal, CustomId, Heading, Alert } from '$lib/components';
    import { createEventDispatcher } from 'svelte';
    import { page } from '$app/stores';
    import { uploader } from '$lib/stores/uploader';
    import { bucket } from './store';
    import { calculateSize } from '$lib/helpers/sizeConvertion';
    import { Permissions } from '$lib/components/permissions';
    import { addNotification } from '$lib/stores/notifications';
    import { trackEvent } from '$lib/actions/analytics';

    export let showCreate = false;

    const bucketId = $page.params.bucket;
    const dispatch = createEventDispatcher();

    let files: FileList;
    let permissions: string[] = [];
    let id: string = null;
    let error: string;
    let showCustomId = false;

    async function create() {
        try {
            showCreate = false;
            await uploader.uploadFile(bucketId, id, files[0], permissions);
            files = null;
            showCustomId = false;
            // uploader.addFile(file);
            dispatch('created');
            addNotification({
                type: 'success',
                message: `File has been uploaded`
            });
            trackEvent('submit_file_create');
        } catch ({ message }) {
            error = message;
        }
    }

    $: if (!showCreate) {
        id = files = error = null;
        permissions = [];
    }
</script>

<Modal size="big" {error} bind:show={showCreate} on:submit={create}>
    <svelte:fragment slot="header">Create File</svelte:fragment>
    <FormList>
        <div>
            <InputFile bind:files allowedFileExtensions={$bucket.allowedFileExtensions} />
            <p>Max file size: {calculateSize($bucket.maximumFileSize)}</p>
        </div>

        {#if !showCustomId}
            <div>
                <Pill button on:click={() => (showCustomId = !showCustomId)}>
                    <span class="icon-pencil" aria-hidden="true" /><span class="text">
                        File ID
                    </span>
                </Pill>
            </div>
        {:else}
            <CustomId bind:show={showCustomId} name="File" bind:id />
        {/if}
        <Heading tag="h6" size="7">Update Permissions</Heading>
        <p class="text">
            Choose who can access your buckets and files. For more information, check out the
            <a
                href="https://appwrite.io/docs/permissions"
                target="_blank"
                rel="noopener noreferrer"
                class="link">Permissions Guide</a> in our documentation.
        </p>
        {#if $bucket.fileSecurity}
            <div class="common-section">
                <Alert type="info">
                    <svelte:fragment slot="title">File security enabled</svelte:fragment>
                    Users will be able to access this file if they have been granted
                    <b>either File or Bucket permissions</b>.
                </Alert>
            </div>
            <div class="common-section">
                <Permissions bind:permissions />
            </div>
        {:else}
            <Alert type="info">
                <svelte:fragment slot="title">File security disabled</svelte:fragment>
                If you want to assign file permissions, navigate to Bucket settings and enable file security.
                Otherwise, only Bucket permissions will be used.
            </Alert>
        {/if}
    </FormList>
    <svelte:fragment slot="footer">
        <Button secondary on:click={() => (showCreate = false)}>Cancel</Button>
        <Button submit>Create</Button>
    </svelte:fragment>
</Modal>
