<script lang="ts">
    import { ExtraButtonComponent } from "obsidian";
    import { SAVE } from "src/utils";

    import type TrackerView from "src/view";
    import { getContext } from "svelte";
    import { createEventDispatcher } from "svelte";

    export let name: string;
    let encounterName = name;
    const view = getContext<TrackerView>("view");

    let saveButton: ExtraButtonComponent;
    const checkSave = () => {
        if (encounterName?.length && saveButton.disabled) {
            saveButton.setDisabled(false);
        } else if (!encounterName?.length && !saveButton.disabled) {
            saveButton.setDisabled(true);
        }
    };
    let checking = false;
    const save = (node: HTMLElement) => {
        saveButton = new ExtraButtonComponent(node)
            .setIcon(SAVE)
            .setDisabled(
                !(encounterName != undefined && encounterName?.length > 0)
            )
            .onClick(async () => {
                if (
                    encounterName &&
                    encounterName in view.plugin.data.encounters &&
                    !checking
                ) {
                    checking = true;
                } else {
                    await view.saveEncounter(encounterName);
                    dispatch("cancel");
                }
            });
    };
    const dispatch = createEventDispatcher();
    const cancel = (node: HTMLElement) => {
        new ExtraButtonComponent(node).setIcon("cross").onClick(() => {
            dispatch("cancel");
        });
    };
</script>

<div class="saving-container">
    <div class="saving-encounter">
        <span>Save encounter as:</span>
        <!-- svelte-ignore a11y-autofocus -->
        <input type="text" bind:value={encounterName} on:input={checkSave} />
    </div>

    <div class="save-buttons">
        {#if checking}
            <span class="checking">
                <small>
                    An encounter by that name already exists. Are you sure?
                </small>
            </span>
        {/if}
        <div class="save" use:save />
        <div class="cancel" use:cancel />
    </div>
</div>

<style>
    .saving-container {
        padding: 0.5rem;
    }
    .saving-encounter {
        display: flex;
        align-items: center;
        justify-content: space-between;
    }
    .save-buttons {
        margin-top: 1rem;
        display: flex;
        justify-content: flex-end;
        gap: 1rem;
    }
    .save-buttons > div {
        display: flex;
        align-items: center;
    }
    .save-buttons :global(.clickable-icon) {
        margin: 0;
    }
    .save-buttons > .save :global(.clickable-icon.is-disabled) {
        cursor: not-allowed;
        color: var(--text-faint);
    }
</style>
