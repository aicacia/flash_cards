<script context="module" lang="ts">
  export async function load(request: {
    page: { params: { deckId: string; cardId: string } };
  }) {
    return {
      props: request.page.params,
    };
  }
</script>

<script lang="ts">
  import { goto } from "$app/navigation";
  import { cardStore, createCard, updateCard } from "$lib/state/cards";
  import Editor from "$lib/Editor.svelte";
  import type { Delta, EditorChangeEvent } from "typewriter-editor";

  export let deckId: string;
  export let cardId: string;

  $: card = $cardStore.table.byId(cardId);
  let loaded = false;
  let front: Delta;
  let back: Delta;

  $: if (card && !loaded) {
    loaded = true;
    front = card.front;
    back = card.back;
  }

  function onFrontChange(event: CustomEvent<EditorChangeEvent>) {
    if (!loaded) {
      return;
    }
    updateCard(cardId, (row) => {
      row.front = event.detail.doc.toJSON();
    });
  }
  function onBackChange(event: CustomEvent<EditorChangeEvent>) {
    if (!loaded) {
      return;
    }
    updateCard(cardId, (row) => {
      row.back = event.detail.doc.toJSON();
    });
  }

  function onCreateCard() {
    const id = createCard(deckId);
    goto(`/decks/${deckId}/cards/${id}/edit`);
  }
</script>

<svelte:head>
  <title>Edit Card {cardId}</title>
</svelte:head>

<div class="mt-2 mb-2">
  <a type="button" class="btn btn-primary" href="{`/decks/${deckId}/edit`}"
    >Back</a>
</div>

<div class="container">
  <div class="row d-flex justify-content-center">
    <div class="col-md-6">
      <h3>Front</h3>
      <Editor delta="{front}" on:change="{onFrontChange}" />
      <hr class="mt-2 mb-2" />
      <h3>Back</h3>
      <Editor delta="{back}" on:change="{onBackChange}" />
    </div>
  </div>
</div>

<div class="mt-2 clearfix">
  <button
    type="button"
    class="btn btn-primary float-end"
    on:click="{onCreateCard}">Create Another Card</button>
</div>
