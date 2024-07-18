<script lang="ts">
    import { createEventDispatcher } from "svelte";

    export let title: string;
    export let cancelButtonText: string = "Cancelar";
    export let acceptButtonText: string = "Aceptar";
    export let accept: (() => void) | null = null;
    export let acceptButtonDisabled: boolean = false;

    const dispatch = createEventDispatcher();
    const CLASS_IS_ACTIVE = "is-active";
    export const EVENT_OPEN = "open";
    export const EVENT_CLOSE = "close";
    export const EVENT_ACCEPT = "accept";
    let modal: Element;

    export function open() {
        if (modal == null) {
            return;
        }

        modal.classList.add(CLASS_IS_ACTIVE);
        dispatch(EVENT_OPEN);
    }

    export function close() {
        if (modal == null) {
            return;
        }

        modal.classList.remove(CLASS_IS_ACTIVE);
        dispatch(EVENT_CLOSE);
    }
</script>

<div class="modal" bind:this={modal}>
    <div class="modal-background" />
    <div class="modal-card">
        <header class="modal-card-head">
            <p class="modal-card-title">{title}</p>

            <button
                class="delete"
                aria-label="close"
                on:click={() => close()}
            />
        </header>

        <section class="modal-card-body">
            <slot />
        </section>

        <footer class="modal-card-foot">
            <button class="button" on:click={() => close()}
                >{cancelButtonText}</button
            >

            {#if accept != null}
                <button
                    class="button is-success"
                    disabled={acceptButtonDisabled}
                    on:click={() => accept && accept()}
                    >{acceptButtonText}</button
                >
            {/if}
        </footer>
    </div>
</div>
