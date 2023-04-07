<script lang="ts">
  type Game = 'waiting for input' | 'in progress' | 'game over';
  type Word = string;

  let game: Game = 'waiting for input';
  let typedLetter = '';

  let words = 'The quick brown fox jumps over a lazy dog'.split(' ');
  let wordIndex = 0;
  let letterIndex = 0;
  let correctLetters = 0;

  let wordsEl: HTMLDivElement;
  let letterEl: HTMLSpanElement;
  let inputEl: HTMLInputElement;
  let caretEl: HTMLDivElement;

  $: if (game === 'in progress') {
    setLetter();
    moveCaret();
  }

  function setGameState(state: Game) {
    game = state;
  }
  function increaseScore() {
    correctLetters = correctLetters + 1;
  }
  function startGame() {
    setGameState('in progress');
  }

  function setLetter() {
    // Before I set the letter I want to check if the word is done
    // by checking if the letterIndex is higher than the length of the word

    const isWordCompleted = letterIndex > words[wordIndex].length - 1;
    if (!isWordCompleted) {
      letterEl = wordsEl.children[wordIndex].children[
        letterIndex
      ] as HTMLSpanElement;
    }
  }

  function checkLetter() {
    // I’m going to compare the typed letter to the current letter.
    // You can get the letter from the words array or wordsEl using innerText on the letter element
    // If the typed letter equals the current letter I’m going to increase the score
    // and set a data attribute data-letter='correct' on the <span> element for styling

    const currentLetter = words[wordIndex][letterIndex];
    if (typedLetter === currentLetter) {
      letterEl.dataset.letter = 'correct';
      increaseScore();
    }
    if (typedLetter !== currentLetter) {
      letterEl.dataset.letter = 'incorrect';
    }
  }

  function nextWord() {
    const isNotFirstLetter = letterIndex !== 0;
    const isOneLetterWord = words[wordIndex].length === 1;

    if (isNotFirstLetter || isOneLetterWord) {
      wordIndex += 1;
      letterIndex = 0;
      increaseScore();
      setLetter();
      moveCaret();
    }
  }

  function nextLetter() {
    letterIndex = letterIndex + 1;
  }

  function resetLetter() {
    // Reset the input
    typedLetter = '';
  }

  function updateLine() {
    const wordEl = wordsEl.children[wordIndex];
    const wordsY = wordsEl.getBoundingClientRect().y;
    const wordY = wordEl.getBoundingClientRect().y;

    if (wordY > wordsY) {
      wordEl.scrollIntoView({ block: 'center' });
    }
  }

  function moveCaret() {
    const offset = 4;
    let topShift = `${letterEl.offsetTop + offset}px`;
    let leftShift =
      letterIndex > 0
        ? `${letterEl.offsetLeft + letterEl.offsetWidth}px`
        : `${letterEl.offsetLeft}px`;
    caretEl.style.top = topShift;
    caretEl.style.left = leftShift;
  }

  function updateGameState() {
    setLetter();
    checkLetter();
    nextLetter();
    resetLetter();
    updateLine();
    moveCaret();
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.code === 'Space') {
      event.preventDefault();
      if (game === 'in progress') nextWord();
    }

    if (game === 'waiting for input') startGame();
  }
</script>

<div class="game" data-game={game}>
  <input
    bind:this={inputEl}
    bind:value={typedLetter}
    on:input={updateGameState}
    on:keydown={handleKeydown}
    class="input"
    type="text"
  />

  <div bind:this={wordsEl} class="words">
    {#each words as word}
      <span class="word">
        {#each word as letter}
          <span class="letter">{letter}</span>
        {/each}
      </span>
    {/each}
    <div bind:this={caretEl} class="caret" />
  </div>
</div>

<style lang="sass">
  .words
    --line-height: 1em
    --lines: 3

    width: 100%
    max-height: calc(var(--line-height) * var(--lines) * 1.42)
    display: flex
    flex-wrap: wrap
    gap: 0.6em
    position: relative
    font-size: 1.5rem
    line-height: var(--line-height)
    overflow: hidden
    user-select: none
    .game
      &[data-game="in progress"] .caret
        animation: none

  .caret
      position: absolute
      height: 1.8rem
      top: 0
      border-right: 1px solid var(--primary)
      animation: caret 1s infinite
      transition: all 0.2s ease

      @keyframes caret
        0%,
        to
          opacity: 0,
        50%
          opacity: 1

  .letter
    opacity: .4
    transition: all 0.3s ease

  :global(.letter[data-letter='correct'])
    opacity: .8 !important

  :global(.letter[data-letter='incorrect'])
    color: var(--primary)
    opacity: 1 !important

</style>
