<script lang="ts">
    import { convertFileSrc, invoke } from '@tauri-apps/api/tauri';
    import { getContext } from 'svelte';
    import { t } from '../../locale/i18n.js';
    import type { IGDBData } from '../../scripts/Interfaces';
    import {
        saveData,
        editData,
        getGameMetadata,
        downloadImage,
    } from '../../scripts/Library';

    import { isEmpty } from "../../scripts/Main.js";

    const { close }: any = getContext('simple-modal');
    export let title: string;
    export let id: number;
    export let executablePath: any;
    export let description: string;
    export let imagePath: any;
    let imageSelected: boolean;
    let moreThanOneGameMeta: boolean;
    let gameMetadata: IGDBData[] = [];
    let gameMetadataModal: HTMLDialogElement;
    
    

    // TS Function -> Rust Function
    // - Opens a File selector dialog
    function chooseExecutable() {
        invoke('file_dialog').then((message) => (executablePath = message));
    }
    function chooseImage() {
        invoke('image_dialog')
            .then((message) => (imagePath = message))
            .then(() => (imageSelected = true));
    }

    export let operationToPerform: string = 'Save';
    async function operation_handler(operation: string) {
        if (operation === 'Save') {
            // Checks if there is no title, description, executable or image
            // -----------------------------------------------------------
            // If the user doesn't select an title, set it as "No title"
            // If the user doesn't select an description set it blank
            // If the user doesn't select an image, set it to "None"
            // And if the user doesn't select an exec, don't allow them to confirm

            if (isEmpty(title)) title = 'No title';
            if (isEmpty(description)) description = '';
            if (isEmpty(executablePath)) return;
            if (isEmpty(imagePath)) imagePath = 'None';

            saveData(title, executablePath, description, imagePath);
            close();
        } else if (operation === 'Edit') {
            // Checks if there are no title, description, exec or img
            // If one of them isn't selected, do not let the user exit
            // Except for the description
            if (isEmpty(title)) return;
            if (isEmpty(description)) description = '';
            if (isEmpty(executablePath)) return;
            if (isEmpty(imagePath)) return;

            editData(id, title, executablePath, description, imagePath);
            close();
        }
    }
</script>

<div class="modal-main">
    <div class="newgame">
        <input
            type="text"
            name="Title"
            placeholder="{$t('modals.newGame.gameTitle')}"
            bind:value="{title}"
            class="title-el"
        />
        <div class="show-path">
            <!-- When the button is clicked, run chooseExecutable -->
            <button on:click="{chooseExecutable}" class="ng-button"
                >{$t('modals.newGame.addExec')}</button
            >

            <!-- Binds the inner html to executablePath -->
            <p
                class="path"
                contenteditable="true"
                bind:innerHTML="{executablePath}"
            ></p>
        </div>
        <div class="show-path">
            <!-- When the button is clicked, run chooseImage -->
            <button on:click="{chooseImage}" class="ng-button image-add"
                >{$t('modals.newGame.addImg')}</button
            >

            <!-- 
				Adds an image preview of the cover art
				Only if imageSelected = true and
				imagePath != "None"
			-->
            {#if imageSelected && imagePath != 'None'}
                <img src="{convertFileSrc(imagePath)}" alt="" width="100px" />
            {/if}
        </div>
        <textarea
            maxlength="800"
            name="Description"
            placeholder="{$t('modals.newGame.desc')}"
            bind:value="{description}"></textarea>

        <button
            class="fetch-meta ng-button"
            on:click="{() => {
                if (!isEmpty(title)) {
                    getGameMetadata(title).then(async (gameMeta) => {
                        gameMetadata = gameMeta;

                        if (gameMeta.length === 0) {
                            invoke('log', {
                                logLevel: 0,
                                logMessage:
                                    'No games found that match the title. Can be a 404 response or an incorrect title',
                            });

                            return;
                        } else if (gameMeta.length <= 1) {
                            description = gameMeta[0].summary;
                            title = gameMeta[0].name;

                            const downImagePath = await downloadImage(
                                gameMeta[0].cover_url
                            );

                            imagePath = downImagePath;

                            return;
                        }

                        invoke('log', {
                            logLevel: 2,
                            logMessage:
                                'More than one result recieved: Opening dialog box',
                        });

                        gameMetadataModal.showModal();
                    });
                    return;
                }

                invoke('log', {
                    logLevel: 0,
                    logMessage: 'No title set',
                });
            }}"
        >
            {$t('modals.newGame.autoFetch')}
        </button>
    </div>

    <dialog class="fetch-meta" bind:this="{gameMetadataModal}">
        <span>{$t('modals.fetchMeta.selectGame')}</span>
        {#each gameMetadata as gameMeta, i}
            <button
                on:click="{async () => {
                    description = gameMetadata[i].summary;
                    title = gameMetadata[i].name;

                    const downImagePath = await downloadImage(
                        gameMetadata[i].cover_url
                    );

                    imagePath = downImagePath;

                    gameMetadataModal.close();
                }}"
            >
                {gameMeta.name}<br />
            </button>
        {/each}
    </dialog>

    <!-- I think you get it by now -->
    <div class="done-btn">
        <button
            on:click="{() => {
                operation_handler(operationToPerform);
            }}"
        >
            {$t('modals.newGame.done')}
        </button>
    </div>
</div>
