<script>
  import { createEventDispatcher, onMount } from "svelte";
  import { createPopper } from "@popperjs/core";

  import { get_caret_coordinates } from "./caret";

  export let placeholder = "Your comments here...";

  let value = "";
  let caret_coordinates = {};
  let height = 200;
  let code;
  let textarea;
  let show = false;
  let popper;
  let timeout = 1000;
  let caret;
  let mention;

  const ITEM_HEIGHT = 35;

  export let item_key = "label";
  export let items = [];

  let index = null;

  const dispatch = createEventDispatcher();

  const handle_on_input = () => {
    dispatch("change", { value });
    calculate_height();
  };

  const handle_on_change = () => {
    dispatch("change", { value });
    calculate_height();
  };

  const get_word_at_cursor = (textarea) => {
    const value = textarea.value;
    const index = textarea.selectionStart;
    const segments = String(value)
      .substring(0, index)
      .split(" ")
      .map((v) => v.trim());

    const word = segments[segments.length - 1];

    return word;
  };

  const read_word_at_cursor = (textarea) => {
    const word = get_word_at_cursor(textarea);

    show = word && word.length > 2 && word.startsWith("@");

    return word;
  };

  const handle_escape = (evt) => {
    if (value && show && items.length) {
      evt.preventDefault();

      value = `${value} `;

      dispatch("change", { value });

      show = false;

      index = null;
    }
  };

  const handle_enter = (evt) => {
    if (value && show && items.length) {
      evt.preventDefault();

      value = `${value.trim()} `;

      dispatch("change", { value });

      const word = get_word_at_cursor(textarea);

      const active = index;

      handle_reset();

      if (typeof active === "number") {
        const params = { value: items[active][item_key], word };

        if (items[active]) {
          handle_select_item(items[active]);
        }

        dispatch("mention", params);
      }
    }
  };

  const handle_reset = (resetIndex = true) => {
    show = false;

    if (resetIndex) index = null;

    textarea.focus();
  };

  const handle_select_item = (item) => {
    const valStr = textarea.value;

    const until = textarea.selectionStart;

    const left = String(valStr)
      .substring(0, until)
      .split(" ")
      .map((v) => v.trim())
      .slice(0, -1)
      .join(" ");

    const right = String(valStr).substring(until, valStr.length);

    const next = left + ` ${item[item_key]}` + `${right} `;

    value = next.trimStart();

    dispatch("change", { value });

    handle_reset();
  };

  const handle_item_click = (item) => {
    if (value && show && items.length) handle_select_item(item);
  };

  const handle_down = (evt) => {
    if (value && show && items.length) {
      evt.preventDefault();

      if (index === null || index === items.length - 1) {
        mention.scroll({
          top: 0,
        });

        index = 0;

        return;
      }

      index = index + 1;

      mention.scroll({
        top: mention.scrollTop + ITEM_HEIGHT,
      });
    }
  };

  const handle_up = (evt) => {
    if (value && show && items.length) {
      evt.preventDefault();

      if (index === 0 || index === null) {
        index = items.length - 1;

        mention.scrollTop = mention.offsetHeight;

        return;
      }

      index = index - 1;

      mention.scroll({
        top: index > 6 ? mention.offsetHeight - (index + 1) * ITEM_HEIGHT : 0,
      });
    }
  };

  const handle_window_keydown = (evt) => {
    const keycode = evt.keyCode;

    switch (keycode) {
      case 27:
        handle_escape(evt);
        break;
      case 13:
        handle_enter(evt);
        break;
      case 38:
        handle_up(evt);
        break;
      case 40:
        handle_down(evt);
        break;
    }
  };

  const handle_on_keyup = (evt) => {
    read_word_at_cursor(textarea);
    calculate_height();
  };

  const handle_on_keydown = (evt) => {
    read_word_at_cursor(textarea);
    calculate_height();
  };

  const create_popper_instance = async (show) => {
    popper = createPopper(caret, mention, {
      placement: "top",
      modifiers: [
        {
          name: "offset",
          options: {
            offset: [100, 30],
          },
        },
      ],
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
  };

  $: show ? create_popper_instance() : destroy_popper_instance();

  $: value && show ? update_popper_instance() : destroy_popper_instance();

  onMount(async () => {
    setTimeout(() => {
      calculate_height();
    }, timeout);
  });
</script>

<svelte:window on:keydown={handle_window_keydown} />

<div class="mention" class:show bind:this={mention}>
  {#each items as item, i}
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
    <div
      id={`mention-item-${index}`}
      class="mention-item"
      tabindex={index}
      class:active={index === i}
      on:click={() => handle_item_click(item)}
    >
      <div class="mention-item-content">
        {item[item_key]}
      </div>
    </div>
  {/each}
</div>

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
    on:keyup={handle_on_keyup}
    on:keydown={handle_on_keydown}
    on:input={handle_on_input}
    on:change={handle_on_change}
    style="min-height: {height}px;"
    bind:this={textarea}
  />
  <code bind:this={code}>
    {value}
  </code>
</div>

<style>
  :root {
    --mention-border-color: #ddd;
    --mention-background: #fff;
    --mention-shadow: -5px 7px 5px -8px rgba(0, 0, 0, 0.75);
  }

  .mention-wrapper {
    border: solid 1px var(--mention-border-color);
    position: relative;
    overflow: hidden;
    font-family: inherit;
  }

  .mention {
    width: 200px;
    max-height: 200px;
    overflow-y: auto;
    position: absolute;
    left: 0;
    top: 0;
    z-index: 2000;
    display: none;
    opacity: 0;
    transition: opacity 300ms ease-in;
    transition: translate 100ms ease-in;

    flex-direction: column;
    border-bottom: solid 1px var(--mention-border-color);
    background-color: var(--mention-background);
    box-shadow: var(--mention-shadow);
  }

  .mention.show {
    display: flex;
    opacity: 1;
  }

  .mention-item {
    display: flex;
    padding: 8px;
    border-bottom: solid 1px var(--mention-border-color);
    transition: all 100ms ease-in;
    cursor: pointer;
  }

  .mention-item.active {
    background-color: aliceblue;
  }

  .mention-item:last-child {
    border-bottom: none;
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
    z-index: 1;
    pointer-events: none;
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
