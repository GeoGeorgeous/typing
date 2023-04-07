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

  function nextLetter() {
    letterIndex = letterIndex + 1;
  }

  function resetLetter() {
    // Reset the input
    typedLetter = '';
  }

  function updateGameState() {
    setLetter();
    checkLetter();
    nextLetter();
    resetLetter();
  }

  function handleKeydown(event: KeyboardEvent) {
    if (event.code === 'Space') event.preventDefault();

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
  </div>
</div>

<style lang="sass">
/* .. */
</style>
