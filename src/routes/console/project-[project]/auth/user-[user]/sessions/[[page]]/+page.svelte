<script lang="ts">
    import { Empty } from '$lib/components';
    import {
        Table,
        TableBody,
        TableHeader,
        TableRow,
        TableCellHead,
        TableCell,
        TableCellText
    } from '$lib/elements/table';
    import { Pill } from '$lib/elements';
    import { Button } from '$lib/elements/forms';
    import { Container } from '$lib/layout';
    import { sdkForProject } from '$lib/stores/sdk';
    import DeleteAllSessions from '../../deleteAllSessions.svelte';
    import DeleteSessions from '../../deleteSession.svelte';
    import type { PageData } from './$types';

    export let data: PageData;

    let showDelete = false;
    let showDeleteAll = false;
    let selectedSessionId: string;

    const getBrowser = (clientCode: string) => sdkForProject.avatars.getBrowser(clientCode, 40, 40);
</script>

<Container>
    {#if data.sessions.total}
        <div class="u-flex u-main-end  common-section">
            <Button secondary on:click={() => (showDeleteAll = true)}>
                <span class="text">Delete All</span>
            </Button>
        </div>
        <Table>
            <TableHeader>
                <TableCellHead width={140}>Browser and Device</TableCellHead>
                <TableCellHead width={140}>Session</TableCellHead>
                <TableCellHead width={140}>Location</TableCellHead>
                <TableCellHead width={140}>IP</TableCellHead>
                <TableCellHead width={30} />
            </TableHeader>
            <TableBody>
                {#each data.sessions.sessions as session}
                    <TableRow>
                        <TableCell title="Client">
                            <div class="u-flex u-gap-12 u-cross-center">
                                <div class="image-item">
                                    <img
                                        height="20"
                                        width="20"
                                        src={getBrowser(session.clientCode).toString()}
                                        alt={session.clientName} />
                                </div>
                                <p class="text">
                                    {session.clientName}
                                    {session.clientVersion} on {session.osName}
                                    {session.osVersion}
                                </p>
                                {#if session.current}
                                    <Pill success>current session</Pill>
                                {/if}
                            </div>
                        </TableCell>

                        <TableCellText title="Session">{session.clientType}</TableCellText>
                        <TableCellText title="Location">{session.countryName}</TableCellText>
                        <TableCellText title="IP">{session.ip}</TableCellText>
                        <TableCellText title="">
                            <button
                                class="button is-only-icon is-text"
                                aria-label="Delete item"
                                on:click={() => {
                                    selectedSessionId = session.$id;
                                    showDelete = true;
                                }}>
                                <span class="icon-trash" aria-hidden="true" />
                            </button>
                        </TableCellText>
                    </TableRow>
                {/each}
            </TableBody>
        </Table>
    {:else}
        <Empty single>
            <p>No session available</p>
            <Button
                external
                secondary
                href="https://appwrite.io/docs/server/auth?sdk=nodejs-default#usersGetSessions">
                Documentation
            </Button>
        </Empty>
    {/if}
    <div class="u-flex u-margin-block-start-32 u-main-space-between">
        <p class="text">Total results: {data.sessions.total}</p>
    </div>
</Container>

<DeleteSessions {selectedSessionId} bind:showDelete />
<DeleteAllSessions bind:showDeleteAll />
