<script lang="ts">
  export let name: string;
</script>

<main>
  <h1>Hello {name}!</h1>
  <p>Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.</p>
</main>

<style>
  main {
      margin: 0 auto;
      padding: 1em;
      max-width: 240px;
      text-align: center;
    }

    h1 {
      color: #ff3e00;
      text-transform: uppercase;
      font-weight: 100;
      font-size: 4em;
    }

    @media (min-width: 640px) {
      main {
        max-width: none;
      }
    }
</style>
