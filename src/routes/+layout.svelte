<script lang="ts">
  import '../styles/app.sass';
</script>

<svelte:head>
  <title>Robo Typing</title>
</svelte:head>

<div class="layout">
  <nav><h1>🦾 Robo Typing</h1></nav>
  <main><slot /></main>
  <footer>
    <p>
      built by <a href="https://github.com/GeoGeorgeous/typing">geo</a
      >
    </p>
    <p>
      / inspired by <a href="https://joyofcode.xyz/svelte-typing-game"
        >joyofcode tutorial</a
      >
    </p>
  </footer>
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

  footer
    opacity: .4
    transition: all 0.3s ease
    display: flex
    gap: 1rem
    &:hover
      opacity: 1
    a
      color: var(--primary)
</style>
