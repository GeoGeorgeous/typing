<script lang="ts">
  import { onMount, tick } from 'svelte';
  import { tweened } from 'svelte/motion';
  import { blur } from 'svelte/transition';

  type Game = 'waiting for input' | 'in progress' | 'game over';
  type Word = string;

  let seconds = 30;
  let toggleReset = false;

  let game: Game = 'waiting for input';
  let typedLetter = '';
  let totalLeters = 0;

  let words: Word[] = [];
  let wordIndex = 0;
  let letterIndex = 0;
  let correctLetters = 0;

  let wordsEl: HTMLDivElement;
  let letterEl: HTMLSpanElement;
  let inputEl: HTMLInputElement;
  let caretEl: HTMLDivElement;
  let playButtonEl: HTMLButtonElement;

  // Results
  let wordsPerMinute = tweened(0, { delay: 300, duration: 1000 });
  let accuracy = tweened(0, { delay: 1300, duration: 1000 });

  // https://www.speedtypingonline.com/typing-equations
  // words per minute = (correct / 5) / time
  // accuracy = (correct / total) * 100%

  function getWordsPerMinute() {
    const word = 5;
    const minutes = 0.5;
    return Math.floor(correctLetters / word / minutes);
  }

  function focusInput() {
    inputEl?.focus();
  }

  function getAccuracy(): number {
    let typedWords: Word[] = words.slice(0, wordIndex);
    let accuracy = 0;
    const isWordTypingInProgress = letterIndex > 0;

    if (isWordTypingInProgress) wordIndex = wordIndex + 1;
    // Count the last word even if the user has not finished it
    typedWords = words.slice(0, wordIndex);

    totalLeters = getTotalLetters(typedWords);

    accuracy = Math.floor((correctLetters / totalLeters) * 100);
    if (isNaN(accuracy)) accuracy = 0;
    return accuracy;
  }

  function getTotalLetters(words: Word[]) {
    const lastWordLettersTyped = words.at(-1)?.slice(0, letterIndex);

    if (lastWordLettersTyped) {
      // Count only part of the last word
      // if the user has not finished it
      return [...words.slice(0, -1), lastWordLettersTyped].reduce(
        (count, word) => count + word.length,
        0
      );
    }

    return words.reduce((count, word) => count + word.length, 0);
  }

  function getResults() {
    $wordsPerMinute = getWordsPerMinute();
    $accuracy = getAccuracy();
  }

  // Reset
  function resetGame() {
    toggleReset = !toggleReset;
    setGameState('waiting for input');
    getWords(100);

    seconds = 30;
    typedLetter = '';
    wordIndex = 0;
    letterIndex = 0;
    correctLetters = 0;
    totalLeters = 0;

    $wordsPerMinute = 0;
    $accuracy = 0;
  }

  function setGameState(state: Game) {
    game = state;
  }

  function increaseScore() {
    correctLetters = correctLetters + 1;
  }

  function decreaseScore() {
    correctLetters = correctLetters - 1;
  }

  function setGameTimer() {
    async function gameTimer() {
      if (seconds > 0) seconds = seconds - 1;

      if (game === 'waiting for input' || seconds === 0) {
        clearInterval(interval);
      }

      if (seconds === 0) {
        setGameState('game over');
        getResults();
        await tick();
        playButtonEl.focus();
      }
    }

    const interval = setInterval(gameTimer, 1000);
  }

  function startGame() {
    setGameState('in progress');
    setGameTimer();
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

  function uncheckLetter() {
    const currentLetter =
      wordsEl.children[wordIndex].children[letterIndex];

    if (letterEl?.dataset.letter) {
      // if user removes the correct letter also decrease the score
      if (letterEl.dataset.letter === 'correct') decreaseScore();
    }
    currentLetter.removeAttribute('data-letter');
  }

  function nextWord() {
    const isNotFirstLetter = letterIndex !== 0;
    const isOneLetterWord = words[wordIndex].length === 1;

    if (isNotFirstLetter || isOneLetterWord) {
      wordIndex += 1;
      letterIndex = 0;
      // Don't count spaces for now
      // increaseScore();
      setLetter();
      moveCaret('start');
    }
  }

  function nextLetter() {
    const lettersInWord = words[wordIndex].length;
    if (letterIndex < lettersInWord) letterIndex = letterIndex + 1;
  }

  function previousLetter() {
    letterIndex = letterIndex - 1;
  }

  function resetLetter() {
    typedLetter = '';
  }

  function goBackspace() {
    const isFirstLetter = letterIndex === 0;
    if (isFirstLetter) return;
    previousLetter();
    setLetter();
    uncheckLetter();
    moveCaret('start');
  }

  function updateLine() {
    const wordEl = wordsEl.children[wordIndex];
    const wordsY = wordsEl.getBoundingClientRect().y;
    const wordY = wordEl.getBoundingClientRect().y;

    if (wordY > wordsY) {
      wordEl.scrollIntoView({ block: 'center' });
    }
  }

  function moveCaret(position: 'start' | 'end' = 'end') {
    const offset = 4;
    let topShift = `${letterEl.offsetTop + offset}px`;

    let leftShift = `${letterEl.offsetLeft}px`;
    if (position === 'end') {
      leftShift = `${letterEl.offsetLeft + letterEl.offsetWidth}px`;
    }

    caretEl.style.top = topShift;
    caretEl.style.left = leftShift;
  }

  function updateGameState() {
    setLetter();
    checkLetter();
    nextLetter();
    resetLetter();
    updateLine();
    moveCaret('end');
  }

  async function handleKeydown(event: KeyboardEvent) {
    const regex = /^[a-zA-Zа-яА-Я ]$/;

    if (event.code === 'Space') {
      event.preventDefault();
      if (game === 'in progress') nextWord();
    }

    if (event.code === 'Backspace') {
      if (game === 'in progress') goBackspace();
    }

    if (event.key.match(regex)) {
      if (game === 'waiting for input') startGame();
      focusInput();
    }
  }

  async function getWords(limit: number) {
    const response = await fetch(`/api/words?limit=${limit}`);
    words = await response.json();
  }

  onMount(() => {
    getWords(100);
    focusInput();
  });
</script>

<svelte:window on:keydown={handleKeydown} />

{#if game !== 'game over'}
  <div class="game" data-game={game}>
    <input
      tabindex="-1"
      bind:this={inputEl}
      bind:value={typedLetter}
      on:input={updateGameState}
      class="input"
      type="text"
    />
    <div class="time">{seconds}</div>

    {#key toggleReset}
      <div in:blur|local bind:this={wordsEl} class="words">
        {#each words as word}
          <span class="word">
            {#each word as letter}
              <span class="letter">{letter}</span>
            {/each}
          </span>
        {/each}
        <div bind:this={caretEl} class="caret" />
      </div>
    {/key}

    <!-- Reset button -->
    <div class="reset">
      <button on:click={resetGame} aria-label="reset">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          viewBox="0 0 24 24"
          width="24"
          height="24"
          stroke-width="1.5"
          stroke="currentColor"
          fill="none"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            d="M15 15l6-6m0 0l-6-6m6 6H9a6 6 0 000 12h3"
          />
        </svg>
      </button>
    </div>
  </div>
{/if}

{#if game === 'game over'}
  <div in:blur class="results">
    <div>
      <p class="title">wpm</p>
      <p class="score">{Math.trunc($wordsPerMinute)}</p>
    </div>

    <div>
      <p class="title">accuracy</p>
      <p class="score">{Math.trunc($accuracy)}%</p>
      <p class="correct">
        <span>{correctLetters}</span> correct of
        <span>{totalLeters}</span>
      </p>
    </div>
    <button on:click={resetGame} class="play" bind:this={playButtonEl}
      >Play again</button
    >
  </div>
{/if}

<style lang="sass">
  .game
    position: relative
    .input
      position: absolute
      opacity: 0
    .time
      position: absolute
      top: -48px
      font-size: 1.5rem
      color: var(--primary)
      opacity: 0
      transition: all 0.3s ease
    &[data-game="in progress"]
      .time
        opacity: 1
    .reset
      width: 100%
      display: grid
      justify-content: center
      margin-top: 2rem

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
    &[data-game="in progress"] .caret
      animation: none

  .caret
      position: absolute
      height: 1.8rem
      top: 0
      border-right: 2px solid var(--primary)
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

  .results
    .title
      border-top: 1pix sloid var(--fg-200)
      font-size: 2rem
      color: var(--fg-200)

    .score
      font-size: 4rem
      color: var(--primary)

    .correct
      font-size: 1rem
      color: var(--fg-200)
      span
        color: var(--primary)

    .play
      border: 1px solid var(--fg-200)
      margin-top: 2rem
</style>
