# Svelte Mention

A mention library for SvelteJS. This enables you to have a facebook-like user mentions in a textareas

![My Image](https://raw.github.com/johndavedecano/svelte-mention/main/screenshot.png)

## Installation

```

npm i svelte-mention

```

```
<script>
  import { Mention } from 'svelte-mention'

  let items = [
    { label: "User User 1" },
    { label: "User User 2" },
    { label: "User User 3" },
    { label: "User User 4" },
    { label: "User User 5" },
    { label: "User User 6" },
    { label: "User User 7" },
    { label: "User User 8" },
    { label: "User User 9" },
    { label: "User User 10" },
    { label: "User User 11" },
    { label: "User User 12" },
  ];

  let value = "How are you today?";

  const handle_change = (evt) => console.log(evt.detail);

  const handle_mention = (evt) => console.log(evt.detail);
</script>

<Mention
  {items}
  on:change={handle_change}
  on:mention={handle_mention}
  bind:value
/>
```

## Customization through CSS Variables

```
--mention-border-color: #ddd;
--mention-background: #fff;
--mention-shadow: -5px 7px 5px -8px rgba(0, 0, 0, 0.75);
--mention-background-hover: aliceblue;
--mention-padding: 8px;
--mention-border-radius: 8px;
--mention-z-index: 2000;
```

## Milestones

- make more customizable
- allow custom component
