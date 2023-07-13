<script lang="ts">
    import {clipboard, Tab, TabGroup} from "@skeletonlabs/skeleton";
    import {fade} from "svelte/transition";

    let selectedTab = 0;

    //disables
    let copied = false;

    //checkboxes
    let dryRun = true;
    let archiveMode = true;
    let compress = true;
    let showProgress = false;
    let showProgress2 = true;
    let partial = false;
    let verbose = false;
    let humanReadable = false;
    let removeSource = false;
    let deleteDestination = false;
    let update = false;

    //inputs
    let sourceServer: null | string = null;
    let sourcePath: null | string = null;
    let destinationPath: null | string = null;

    //bottom inputs
    let bandwidthLimit: null | number = null;
    let includeFiles: null | string = null;
    let excludeFiles: null | string = null;
    let fileSizeLimit: null | number = null;
    let sshPort: null | number = 22;

    // output
    let output: null | string = null;


    $: {
        let rsyncCommand = 'rsync ';
        if (sourceServer && sourcePath && destinationPath) {
            rsyncCommand += dryRun ? '--dry-run ' : '';
            rsyncCommand += archiveMode ? '--archive ' : '';
            rsyncCommand += compress ? '--compress ' : '';
            rsyncCommand += showProgress ? '--progress ' : '';
            rsyncCommand += showProgress2 ? '--info=progress2 ' : '';
            rsyncCommand += verbose ? '--verbose ' : '';
            rsyncCommand += humanReadable ? '--human-readable ' : '';
            rsyncCommand += removeSource ? '--remove-source-files ' : '';
            rsyncCommand += deleteDestination ? '--delete ' : '';
            rsyncCommand += update ? '--update ' : '';

            if (sshPort && sshPort !== 22) {
                // Specify the ssh configuration command, assuming user wants to use a custom ssh port 2222
                rsyncCommand += `-e "ssh -p ${sshPort}" `;
            }

            if (bandwidthLimit) {
                rsyncCommand += `--bwlimit=${bandwidthLimit} `;
            }

            if (includeFiles) {
                // multiple files can be separated by comma
                const files = includeFiles.split(',');
                files.forEach(file => rsyncCommand += `--include=${file.trim()} `);
            }

            if (excludeFiles) {
                // multiple files can be separated by comma
                const files = excludeFiles.split(',');
                files.forEach(file => rsyncCommand += `--exclude=${file.trim()} `);
            }

            if (fileSizeLimit) {
                rsyncCommand += `--max-size=${fileSizeLimit}K `;
            }

            let source = sourcePath || 'source_directory';
            let destination = destinationPath || 'destination_directory';

            // if selectedTab is 0, pull from the remote source
            // if selectedTab is 1, rsync to the remote source
            if (selectedTab === 0) {
                source = `${sourceServer}:${source}`;
            } else if (selectedTab === 1) {
                destination = `${sourceServer}:${destination}`;
            }

            rsyncCommand += `${source} ${destination}`;
            output = rsyncCommand;
        } else {
            output = null;
        }
    }

    let timeoutId;

    function toggleCopyNotification() {
        copied = true;

        if (timeoutId) {
            clearTimeout(timeoutId);
        }

        timeoutId = setTimeout(() => {
            copied = false;
        }, 1200);
    }
</script>

<div class="bg-surface-100-800-token p-8 mb-3">
    <TabGroup
            justify="justify-center"
            active="variant-ghost-primary"
            hover="hover:variant-soft-primary"
            flex="flex-1"
            rounded=""
            border=""
            class="bg-surface-100-800-token w-full"
    >
        <Tab bind:group={selectedTab} name="tab1" value={0}>
            <svelte:fragment slot="lead">
                <svg fill="none" class="inline w-5 h-5" stroke="currentColor" stroke-width="1.5" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M12 9.75v6.75m0 0l-3-3m3 3l3-3m-8.25 6a4.5 4.5 0 01-1.41-8.775 5.25 5.25 0 0110.233-2.33 3 3 0 013.758 3.848A3.752 3.752 0 0118 19.5H6.75z"></path>
                </svg>
            </svelte:fragment>
            <span>Remote Source</span>
        </Tab>
        <Tab bind:group={selectedTab} name="tab2" value={1}>
            <svelte:fragment slot="lead">
                <svg fill="none" class="inline w-5 h-5" stroke="currentColor" stroke-width="1.5" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M12 16.5V9.75m0 0l3 3m-3-3l-3 3M6.75 19.5a4.5 4.5 0 01-1.41-8.775 5.25 5.25 0 0110.233-2.33 3 3 0 013.758 3.848A3.752 3.752 0 0118 19.5H6.75z"></path>
                </svg>
            </svelte:fragment>
            <span>Remote Destination</span>
        </Tab>
        <!-- Tab Panels --->
        <svelte:fragment slot="panel">
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-5">
                <div class="flex w-full items-center" class:order-2={selectedTab === 1}>
                    <label class="label">
                        <span class="text-surface-400">{#if selectedTab === 0}Source Server{:else if selectedTab === 1}Destination Server{/if}</span>
                        <input bind:value={sourceServer} class="input inline" type="text" placeholder="username@remote_host"/>
                    </label>
                    <span class="p-2 block mt-5">:</span>
                    <label class="label">
                        <span class="text-surface-400">{#if selectedTab === 0}Source Path{:else if selectedTab === 1}Destination Path{/if}</span>
                        <input bind:value={sourcePath} class="input inline" type="text" placeholder="/var/www/example/"/>
                    </label>
                </div>

                <div class="flex ">
                    <label class="label w-full">
                        <span class="text-surface-400">{#if selectedTab === 0}Destination Path{:else if selectedTab === 1}Source Path{/if}</span>
                        <input bind:value={destinationPath} class="input inline" type="text" placeholder="/var/www/example/"/>
                    </label>
                </div>
            </div>
        </svelte:fragment>
        <!-- ... -->
    </TabGroup>

    <hr class="my-5">

    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 w-full gap-5">
        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={dryRun}/>
            <div>
                <p>Dry Run</p>
                <span class="text-sm text-surface-400">Show what would have been transferred</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={archiveMode}/>
            <div>
                <p>Archive mode</p>
                <span class="text-sm text-surface-400">Same as -rlptgoD</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={compress}/>
            <div>
                <p>Compress</p>
                <span class="text-sm text-surface-400">Compress file data during the transfer</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input on:click={() => showProgress2 = false} class="checkbox" type="checkbox" bind:checked={showProgress}/>
            <div>
                <p>Show Progress</p>
                <span class="text-sm text-surface-400">Show progress during transfer</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input on:click={() => showProgress = false} class="checkbox" type="checkbox" bind:checked={showProgress2}/>
            <div>
                <p>Show Progress 2</p>
                <span class="text-sm text-surface-400">Option shows statistics based on the whole transfer.</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={partial}/>
            <div>
                <p>Partial</p>
                <span class="text-sm text-surface-400">Keep partially transferred files</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={verbose}/>
            <div>
                <p>Verbose</p>
                <span class="text-sm text-surface-400">Increase verbosity</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={humanReadable}/>
            <div>
                <p>Human Readable</p>
                <span class="text-sm text-surface-400">Output numbers in a human-readable format</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={removeSource}/>
            <div>
                <p>Remove Source</p>
                <span class="text-sm text-surface-400">Sender removes synchronized files (non-dirs)</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={deleteDestination}/>
            <div>
                <p>Delete</p>
                <span class="text-sm text-surface-400">Delete extraneous files from destination dirs</span>
            </div>
        </label>

        <label class="flex items-center space-x-2 select-none cursor-pointer">
            <input class="checkbox" type="checkbox" bind:checked={update}/>
            <div>
                <p>Update</p>
                <span class="text-sm text-surface-400">Skip files that are newer on the receiver</span>
            </div>
        </label>
    </div>

    <hr class="my-5">

    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-5">
        <label class="label">
            <span>Limit bandwidth in KB/s</span>
            <input class="input" bind:value={bandwidthLimit} type="number" min="0" placeholder="e.g. 500"/>
        </label>

        <label class="label">
            <span>Include files</span>
            <input class="input" bind:value={includeFiles} type="text" placeholder="e.g. git/* , media/*"/>
        </label>

        <label class="label">
            <span>Exclude files</span>
            <input class="input" bind:value={excludeFiles} type="text" placeholder="e.g. vendor/*"/>
        </label>

        <label class="label">
            <span>File size limit</span>
            <input class="input" bind:value={fileSizeLimit} type="number" min="0" placeholder="e.g. 500"/>
        </label>

        <label class="label">
            <span>SSH Port</span>
            <input class="input" bind:value={sshPort} type="number" min="0" placeholder="22"/>
        </label>
    </div>
</div>

{#if output}
    <div class="flex justify-center mb-3">
        <div use:clipboard={output} on:click={toggleCopyNotification} class="card p-5 cursor-pointer hover:variant-ghost-primary transition" class:hover:variant-ghost-success={copied} class:variant-ghost-success={copied}>
            <h3>
                {#if copied}
                    <svg in:fade={{duration: 200}} fill="none" stroke="currentColor" class="w-5 h-5 inline mr-1.5" stroke-width="1.5" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M11.35 3.836c-.065.21-.1.433-.1.664 0 .414.336.75.75.75h4.5a.75.75 0 00.75-.75 2.25 2.25 0 00-.1-.664m-5.8 0A2.251 2.251 0 0113.5 2.25H15c1.012 0 1.867.668 2.15 1.586m-5.8 0c-.376.023-.75.05-1.124.08C9.095 4.01 8.25 4.973 8.25 6.108V8.25m8.9-4.414c.376.023.75.05 1.124.08 1.131.094 1.976 1.057 1.976 2.192V16.5A2.25 2.25 0 0118 18.75h-2.25m-7.5-10.5H4.875c-.621 0-1.125.504-1.125 1.125v11.25c0 .621.504 1.125 1.125 1.125h9.75c.621 0 1.125-.504 1.125-1.125V18.75m-7.5-10.5h6.375c.621 0 1.125.504 1.125 1.125v9.375m-8.25-3l1.5 1.5 3-3.75"></path>
                    </svg>
                {:else}
                    <svg in:fade={{duration: 200}} fill="none" class="w-5 h-5 inline mr-1.5" stroke="currentColor" stroke-width="1.5" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M15.666 3.888A2.25 2.25 0 0013.5 2.25h-3c-1.03 0-1.9.693-2.166 1.638m7.332 0c.055.194.084.4.084.612v0a.75.75 0 01-.75.75H9a.75.75 0 01-.75-.75v0c0-.212.03-.418.084-.612m7.332 0c.646.049 1.288.11 1.927.184 1.1.128 1.907 1.077 1.907 2.185V19.5a2.25 2.25 0 01-2.25 2.25H6.75A2.25 2.25 0 014.5 19.5V6.257c0-1.108.806-2.057 1.907-2.185a48.208 48.208 0 011.927-.184"></path>
                    </svg>
                {/if}
                {output}
            </h3>
        </div>
    </div>
{/if}