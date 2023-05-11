<script>
  import Mention from "$lib/Mention.svelte";

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
