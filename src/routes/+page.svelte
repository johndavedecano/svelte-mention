<script>
  import { onMount } from "svelte";
  import { createPopper } from "@popperjs/core";

  import { get_caret_coordinates } from "$lib/caret";

  export let placeholder = "Your comments here...";

  let value = "";
  let caret_coordinates = { top: 0, left: 0, height: 0, width: 0 };
  let height = 200;
  let code;
  let textarea;
  let show = false;
  let popper;
  let timeout = 1000;
  let caret;
  let mention;

  const delay_action = () => {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve();
      }, timeout);
    });
  };

  const parse_current_word = () => {};

  const create_popper_instance = async (show) => {
    await delay_action();

    popper = createPopper(caret, mention, {
      placement: "auto",
    });
  };

  const destroy_popper_instance = () => {
    if (popper && typeof popper.destroy === "function") {
      popper.destroy();
    }
  };

  const update_popper_instance = () => {
    if (popper && typeof popper.update === "function") {
      popper.update();
    }
  };

  const calculate_height = () => {
    height = code.offsetHeight;

    caret_coordinates = get_caret_coordinates(textarea);

    if (value.length > 20) show = true;
  };

  $: show ? create_popper_instance() : destroy_popper_instance();

  $: value && show ? update_popper_instance() : destroy_popper_instance();

  onMount(async () => {
    setTimeout(() => {
      calculate_height();
    }, timeout);
  });
</script>

{#if show}
  <div class="mention" bind:this={mention}>testest</div>
{/if}

<span
  bind:this={caret}
  style="left: {caret_coordinates.left}px; top: {caret_coordinates.top}px;"
/>

<div
  class="mention-wrapper"
  style="height: {height}px; --min-height: {height}px"
>
  <textarea
    {placeholder}
    bind:value
    on:keyup={calculate_height}
    on:keydown={calculate_height}
    on:keypress={calculate_height}
    on:input={calculate_height}
    on:change={calculate_height}
    style="min-height: {height}px;"
    bind:this={textarea}
  />
  <code bind:this={code}>
    {value}
  </code>
</div>

<style>
  .mention-wrapper {
    border: solid 1px #ddd;
    position: relative;
    overflow: hidden;
  }

  .mention {
    width: 200px;
    height: 200px;
    background-color: black;
    position: absolute;
    left: 0;
    top: 0;
    z-index: 2000;
  }

  span {
    position: absolute;
    width: 1px;
    height: 1px;
    left: 0;
    top: 0;
  }

  code {
    width: 100%;
    min-height: var(--min-height);
    display: block;
    white-space: pre-wrap;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    z-index: -1;
    color: black;
  }

  textarea {
    width: 100%;
    height: 100%;
    min-height: var(--min-height);
    border: none;
    padding: 0;
    margin: 0;
    display: block;
    box-sizing: border-box;
    resize: none;
    overflow: hidden;
    left: 0;
    right: 0;
    bottom: 0;
    top: 0;
    z-index: 3;
  }

  textarea:focus {
    outline: none;
  }
</style>
