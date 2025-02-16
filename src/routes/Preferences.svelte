<script lang="ts">
    import translations from '../locale/locales.js';
    import { dict, locale, t } from '../locale/i18n.js';
    import {
        installPlugin,
        saveData,
        uninstallPlugin,
        resetTheme,
        wipeLibrary,
    } from '../scripts/Preferences.js';
    import languageNames from '../locale/languages.json';
    import { getPlugins } from '../scripts/Browse.js';
    import {
        Cube,
        TrashBin,
        Albums,
        OpenOutline,
        Close,
        CloudDownload,
        CloudUpload,
    } from 'svelte-ionicons';
    import { toast } from '@zerodevx/svelte-toast';
    import { convertFileSrc } from '@tauri-apps/api/tauri';
    const plugins = getPlugins();

    $: languages = Object.keys(translations);
    $: dict.set(translations);

    let stylesheetUrl: string;
    let updaterStatus: boolean;
</script>

<main class="container">
    <div class="main">
        <div class="section">
            <div class="plugin-card">
                <div class="header">
                    <span>{$t('pluginText')}</span>
                    <Cube size="18px" />
                </div>
                <div class="buttons">
                    <button
                        id="install"
                        on:click="{() =>
                            installPlugin().then((res) => {
                                if (res === 0) {
                                    toast.push('Successfully installed plugin');
                                } else if (res === 1) {
                                    toast.push('Plugin installation failed');
                                }
                            })}"
                    >
                        <Cube class="cube" size="18px" />
                        {$t('preferences.installPlugin')}
                    </button>
                    <button id="wipe" on:click="{wipeLibrary}">
                        <TrashBin class="bin" size="18px" />
                        {$t('preferences.wipeLibrary')}
                    </button>
                </div>
            </div>

            <div class="plugin-card">
                <div class="header">
                    <span>{$t('themeText')}</span>
                    <Albums size="18px" />
                </div>

                <div class="buttons">
                    <input
                        type="text"
                        class="input"
                        placeholder="Insert a theme URL"
                        bind:value="{stylesheetUrl}"
                    />

                    <button
                        on:click="{() =>
                            resetTheme().then(() => {
                                stylesheetUrl = '';
                            })}">Reset to default</button
                    >
                </div>
            </div>

            <div class="plugin-card">
                <div class="header">
                    <span>Updater</span>
                    <CloudDownload size="18px" />
                </div>

                <div class="checkbox">
                    <input type="checkbox" bind:checked="{updaterStatus}" />
                    <p>Supress updater</p>
                </div>
            </div>

            <div class="plugin-card">
                <div class="header">
                    <span>{$t('languageText')}</span>
                    <Cube size="18px" />
                </div>
                <div class="buttons">
                    <select bind:value="{$locale}">
                        {#each languages as languageCode, i}
                            <option value="{languageCode}">
                                {languageNames[i]}
                            </option>
                        {/each}
                    </select>
                </div>
            </div>

            <div class="plugin-card">
                <div class="header">
                    <span>Save</span>
                    <CloudUpload size="18px" />
                </div>

                <div class="buttons">
                    <button
                        class="save-button"
                        on:click="{() =>
                            saveData(
                                $locale,
                                updaterStatus,
                                stylesheetUrl.startsWith(
                                    'https://raw.githubusercontent'
                                ) || stylesheetUrl === undefined
                                    ? stylesheetUrl
                                    : convertFileSrc(stylesheetUrl)
                            )}"
                    >
                        {$t('preferences.saveText')}
                    </button>
                </div>
            </div>
        </div>

        <div class="available-plugins">
            <div class="header">
                <span>{$t('preferences.availablePlugins')}</span>
                <Cube size="18px" />
            </div>
            <ul class="cards">
                {#await plugins then plugins}
                    {#each plugins as plugin}
                        <div class="card">
                            <div class="card-left">
                                <p class="card-header">
                                    {plugin.name}
                                    <a
                                        href="{plugin.source}"
                                        target="_blank"
                                        rel="noreferrer"
                                        title="Open plugin source"
                                    >
                                        <OpenOutline size="20px" />
                                    </a>
                                </p>
                                <div class="card-footer">
                                    <span class="author" title="Plugin author">
                                        <b>
                                            {$t(
                                                'preferences.pluginCard.author'
                                            )}
                                        </b>
                                        {plugin.author}
                                    </span>
                                    <span class="desc">
                                        <b>
                                            {$t('preferences.pluginCard.desc')}
                                        </b>
                                        {plugin.description}
                                    </span>
                                </div>
                            </div>

                            <div class="buttons">
                                <button
                                    class="remove"
                                    on:click="{() => uninstallPlugin(plugin)}"
                                >
                                    <Close size="22px" />
                                </button>
                            </div>
                        </div>
                    {/each}
                    {#if plugins.length === 0}
                        <h3 class="noresults">None yet...</h3>
                    {/if}
                {/await}
            </ul>
        </div>
    </div>
</main>
