<script lang="ts">
  import { onMount } from 'svelte';
  import '../styles/app.sass';

  let darkMode = true;

  function toggleTheme() {
    darkMode = !darkMode;
    localStorage.setItem('dark-mode', darkMode.toString());
    applyTheme();
  }

  function applyTheme() {
    const theme = darkMode ? 'dark' : 'light';
    document.documentElement.setAttribute('data-theme', theme);
  }

  onMount(() => {
    darkMode = localStorage.getItem('dark-mode') === 'true' || false;

    applyTheme();
  });
</script>

<svelte:head>
  <title>Robo Typing</title>
  <script>
    const theme =
      localStorage.getItem('dark-mode') === 'true' ? 'dark' : 'light';
    document.documentElement.setAttribute('data-theme', theme);
  </script>
</svelte:head>

<div class="layout">
  <nav>
    <h1>🦾 Robo Typing</h1>
    <button class="theme-toggle" tabindex="0" on:click={toggleTheme}
      >{darkMode ? 'Light' : 'Dark'}mode</button
    >
  </nav>
  <main><slot /></main>
  <footer>
    <i class="ph-bold ph-bold ph-github-logo" />
    <p>
      built by
      <a href="https://github.com/GeoGeorgeous/typing">geo</a>
    </p>
  </footer>
</div>

<style lang="sass">
   h1
    font-family: 'Lexend Deca', sans-serif
    font-size: 1.8rem
    color: var(--fg-200)
    letter-spacing: 2px

  nav
    display: flex
    align-items: baseline
    flex-flow: row nowrap
    justify-content: space-between

  .layout
    height: 100%
    display: grid
    grid-template-rows: auto 1fr
    align-items: center
    padding: 2rem

  .theme-toggle
    border-radius: 5px
    padding: 0.4rem
    border: 1px solid var(--fg-200)
    max-width: max-content
    &:hover
      outline: 3px solid var(--primary)

  footer
    display: flex
    gap: .4rem
    align-items: baseline
    transition: opacity .3s ease
    p
      opacity: 0.4
      &:has(a:focus, a:hover)
        opacity: 1
    &:has(a:focus, a:hover)
      i
        animation: flicker 5s linear infinite
    a
      text-decoration: none
      border-radius: 5px
      padding:  0.4rem 0.1rem
      color: var(--primary)
      &:focus
        outline: 3px solid var(--primary)

  i
    color: var(--primary)

  @keyframes flicker
    0%, 19.999%, 22%, 62.999%, 64%, 64.999%, 70%, 100%
      opacity: .99

    20%, 21.999%, 63%, 63.999%, 65%, 69.999%
      opacity: 0.4




</style>
