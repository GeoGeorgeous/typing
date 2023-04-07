<script lang="ts">
  import '../styles/app.sass';
</script>

<svelte:head>
  <title>Typing</title>
</svelte:head>

<div class="layout">
  <nav><h1>🔥 typing</h1></nav>
  <main><slot /></main>
</div>

<style lang="sass">
   h1
    font-family: 'Lexend Deca', sans-serif
    font-size: 1.8rem
    color: var(--fg-200)
    letter-spacing: 2px


  .layout
    height: 100%
    display: grid
    grid-template-rows: auto 1fr
    align-items: center
    padding: 2rem

</style>
