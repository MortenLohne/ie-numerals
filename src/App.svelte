<script>
  import { languages } from './lib/languages.js';

  let selectedLanguage = '';
  let gameNumbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  let shuffledWords = [];
  let gridSlots = Array(10).fill(null);
  let draggedWord = null;
  let draggedFromSlot = null;
  let gameWon = false;
  let slotFeedback = Array(10).fill(null); // null: not checked, true: correct, false: incorrect
  let hasSubmitted = false;

  // Initialize game
  function startGame() {
    gameWon = false;
    hasSubmitted = false;
    slotFeedback = Array(10).fill(null);
    shuffledWords = [];

    // Pick a random language
    const languageKeys = Object.keys(languages);
    selectedLanguage = languageKeys[Math.floor(Math.random() * languageKeys.length)];

    const words = gameNumbers.map(num => ({
      number: num,
      word: languages[selectedLanguage].numbers[num]
    }));
    // Shuffle the words and place them directly in the grid
    const shuffled = [...words].sort(() => Math.random() - 0.5);
    gridSlots = [...shuffled];
  }

  // Drag and drop handlers
  function handleDragStart(event, word, fromSlot = null) {
    draggedWord = word;
    draggedFromSlot = fromSlot;
    event.dataTransfer.effectAllowed = 'move';
  }

  function handleDragOver(event) {
    event.preventDefault();
    event.dataTransfer.dropEffect = 'move';
  }

  function handleDrop(event, slotIndex) {
    event.preventDefault();

    if (draggedWord && draggedFromSlot !== null) {
      const targetWord = gridSlots[slotIndex];

      // If target slot has a word, swap them
      if (targetWord) {
        gridSlots[draggedFromSlot] = targetWord;
      } else {
        // Target slot is empty, just clear the source slot
        gridSlots[draggedFromSlot] = null;
      }

      // Place the dragged word in the target slot
      gridSlots[slotIndex] = draggedWord;
      draggedWord = null;
      draggedFromSlot = null;
    }
  }

  function handleSubmit() {
    hasSubmitted = true;

    // Check each slot
    slotFeedback = gridSlots.map((word, index) => {
      if (!word) return null; // Empty slot
      const expectedNumber = index < 9 ? index + 1 : 10;
      return word.number === expectedNumber;
    });

    // Check if all are correct
    const allCorrect = slotFeedback.every(feedback => feedback === true);
    if (allCorrect) {
      gameWon = true;
    } else {
      // If not all correct, clear feedback after 3 seconds
      setTimeout(() => {
        slotFeedback = Array(10).fill(null);
        hasSubmitted = false;
      }, 3000);
    }
  }

  // Start game on mount
  startGame();
</script>

<main>
  <h1>Guess the numbers</h1>

  <div class="language-display">
    Language: <strong>{languages[selectedLanguage]?.name || 'Loading...'}</strong>
  </div>

  <div class="controls">
    <button class="submit-button" on:click={handleSubmit}>Submit Answer</button>
    <button on:click={startGame}>New Game</button>
  </div>

  {#if gameWon}
    <div class="win-message">
      🎉 Congratulations! You got them all right!
    </div>
  {/if}

  <div class="game-container">
    <div class="grid-container">
      <div class="grid">
        {#each Array(9) as _, i}
          <div
            class="grid-slot"
            class:correct={hasSubmitted && slotFeedback[i] === true}
            class:incorrect={hasSubmitted && slotFeedback[i] === false}
            on:dragover={handleDragOver}
            on:drop={(e) => handleDrop(e, i)}
          >
            <div class="slot-number">{i + 1}</div>
            {#if gridSlots[i]}
              <div
                class="word-card draggable"
                draggable="true"
                on:dragstart={(e) => handleDragStart(e, gridSlots[i], i)}
              >
                {gridSlots[i].word}
              </div>
            {/if}
          </div>
        {/each}
      </div>

      <div class="bottom-row">
        <div
          class="grid-slot slot-10"
          class:correct={hasSubmitted && slotFeedback[9] === true}
          class:incorrect={hasSubmitted && slotFeedback[9] === false}
          on:dragover={handleDragOver}
          on:drop={(e) => handleDrop(e, 9)}
        >
          <div class="slot-number">10</div>
          {#if gridSlots[9]}
            <div
              class="word-card draggable"
              draggable="true"
              on:dragstart={(e) => handleDragStart(e, gridSlots[9], 9)}
            >
              {gridSlots[9].word}
            </div>
          {/if}
        </div>
      </div>
    </div>
  </div>
</main>

<style>
  main {
    max-width: 800px;
    margin: 0 auto;
    padding: 2rem;
    font-family: Arial, sans-serif;
  }

  h1 {
    text-align: center;
    color: #333;
    font-size: 1.5rem;
    margin: 0.5rem 0;
  }

  .language-display {
    text-align: center;
    font-size: 1.1rem;
    color: #555;
    margin-bottom: 1rem;
  }

  .language-display strong {
    color: #2196F3;
  }

  .controls {
    display: flex;
    gap: 1rem;
    justify-content: center;
    align-items: center;
    margin-bottom: 2rem;
  }

  button {
    padding: 0.5rem 1rem;
    font-size: 1rem;
    border-radius: 4px;
    border: 1px solid #ccc;
    cursor: pointer;
  }

  button {
    background-color: #4CAF50;
    color: white;
    border: none;
  }

  button:hover {
    background-color: #45a049;
  }

  .submit-button {
    background-color: #2196F3;
  }

  .submit-button:hover {
    background-color: #1976D2;
  }

  .win-message {
    background-color: #4CAF50;
    color: white;
    padding: 1rem;
    border-radius: 8px;
    text-align: center;
    margin-bottom: 1rem;
    font-size: 1.2rem;
  }

  .game-container {
    display: flex;
    flex-direction: column;
    gap: 2rem;
  }

  .grid-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.5rem;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 0.5rem;
    width: fit-content;
  }

  .bottom-row {
    display: flex;
    justify-content: center;
  }

  .grid-slot {
    width: 130px;
    height: 130px;
    border: 2px solid #ddd;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #f9f9f9;
    position: relative;
    transition: background-color 0.5s ease, border-color 0.5s ease, border-width 0.5s ease;
  }

  .grid-slot:hover {
    background-color: #e8f5e9;
    border-color: #4CAF50;
  }

  .grid-slot.correct {
    background-color: #c8e6c9;
    border-color: #4CAF50;
    border-width: 3px;
  }

  .grid-slot.incorrect {
    background-color: #ffcdd2;
    border-color: #f44336;
    border-width: 3px;
  }

  .slot-number {
    font-size: 5rem;
    color: #e0e0e0;
    font-weight: bold;
    position: absolute;
    pointer-events: none;
    z-index: 0;
  }

  .word-card {
    color: #333;
    cursor: move;
    user-select: none;
    font-weight: 600;
    font-size: 1.5rem;
    position: absolute;
    z-index: 1;
    text-align: center;
    padding: 0.5rem;
  }

  .word-card:hover {
    color: #000;
  }

  .word-card:active {
    cursor: grabbing;
  }

  @media (max-width: 600px) {
    main {
      padding: 1rem;
    }

    h1 {
      font-size: 1.2rem;
      margin: 0.25rem 0;
    }

    .language-display {
      font-size: 0.9rem;
      margin-bottom: 0.5rem;
    }

    .controls {
      margin-bottom: 1rem;
    }

    .game-container {
      padding: 0.5rem;
    }

    .grid-slot {
      width: 90px;
      height: 90px;
    }

    .slot-number {
      font-size: 3.5rem;
    }

    .word-card {
      font-size: 1.1rem;
    }
  }
</style>
