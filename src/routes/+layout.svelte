<script lang="ts">
  import { onMount } from 'svelte';
  import '../styles/app.sass';
  import { loop_guard } from 'svelte/internal';

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
    <button class="theme-toggle" on:click={toggleTheme}
      >Go {darkMode ? 'light' : 'dark'}</button
    >
  </nav>
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

  nav
    display: flex
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
