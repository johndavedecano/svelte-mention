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
    {
      label: 'Option 1'
    },
    {
      label: 'Option 2'
    },
    {
      label: 'Option 3'
    }
  ]
  
  const on_change = evt => console.log(evt.detail.value)
</script>

<Mention {items} on:change={on_change} />
```

## Milestones
- searchable items
- make more customizable
- allow custom component
