<script>
  import Meld from './Meld.svelte';

  // Instantiate array tracking structures for your 4 primary melds
  let melds = [
    { type: "CHI", isSimple: true, isOpen: false, isWait: false },
    { type: "CHI", isSimple: true, isOpen: false, isWait: false },
    { type: "CHI", isSimple: true, isOpen: false, isWait: false },
    { type: "CHI", isSimple: true, isOpen: false, isWait: false }
  ];

  // Independent isolated tracking logic for the final pair structure
  let pairType = "WIND"; // DRAGON, WIND, SIMPLE
  let pairIsWait = false;

  let isTsumo = false;
  let base = 20;
  if (isTsumo) { base += 2};

  // Reactively tally up all your separate Fu calculations automatically
  $: totalFu = calculateTotalFu(melds, pairType, pairIsWait, isTsumo);

  function calculateTotalFu(currentMelds, pair, wait, tsumo) {
    let base = 20;
    // Add logic summing values here...
    return base;
  }
</script>

<main class="app-container">
  <header>
    <h1>Just Count Fu</h1>
    <p class="subtitle">Riichi Fu Calculator</p>
  </header>

  <!-- High level layout automatically converting to rows on cramped mobile screens -->
  <div class="board-layout">
    {#each melds as meld, index}
      <Meld
        bind:type={meld.type}
        bind:isSimple={meld.isSimple}
        bind:isOpen={meld.isOpen}
        bind:isWait={meld.isWait}
      />
    {/each}

    <!-- Simpler manual pair controller card styling -->
    <div class="pair-card" class:active-wait={pairIsWait}>
      <div class="toggle-group white">
        <label><input type="radio" bind:group={pairType} value="DRAGON" /><span>DRAGON</span></label>
        <label><input type="radio" bind:group={pairType} value="WIND" /><span>WIND</span></label>
      </div>
      <div class="tile-display">
        <div class="tile">#</div>
        <div class="tile">#</div>
      </div>
      <div class="info-footer">
        <div class="fu-display">[0]</div>
        <button class="wait-btn" class:waiting={pairIsWait} on:click={() => pairIsWait = !pairIsWait}>
          WAIT
        </button>
      </div>
    </div>
  </div>

  <!-- Bottom Global Action Controls -->
  <footer>
    <div class="toggle-group orange narrow">
      <label><input type="radio" bind:group={isTsumo} value={false} /><span>RON</span></label>
      <label><input type="radio" bind:group={isTsumo} value={true} /><span>TSUMO</span></label>
    </div>

    <div class="results">
      <p>base: [{base}]</p>
      <p class="final-score">Score is {totalFu} -> ({totalFu})</p>
      <button class="reset-btn">RESET</button>
    </div>
  </footer>
</main>

<style>
  :global(body) { background-color: #121212; color: #fff; margin: 0; }
  .app-container { max-width: 1600px; margin: 0 auto; padding: 16px; font-family: sans-serif; }

  h1 { font-size: 1.2rem; margin: 0; text-align: center; color: #eee; }
  .subtitle { font-size: 0.9rem; color: #777; margin: 2px 0 16px 0; }

  .board-layout {
    display: flex;
    flex-direction: row;       /* Force horizontal alignment by default */
    flex-wrap: wrap;           /* Safely wrap to the next line if the screen is too narrow */
    justify-content: center;   /* Center the items horizontally on the screen */
    gap: 16px;                 /* Clean spacing between the card groups */
    width: 100%;
    margin-bottom: 24px;
  }

  .pair-card {
    background: #1e1e1e; padding: 12px; border-radius: 8px;
    display: flex; flex-direction: column; align-items: center; gap: 8px;
    border: 2px solid transparent;
  }
  .active-wait { border-color: #0284c7; }

  /* Re-use global toggle design styles */
  .toggle-group { display: flex; background: #333; border-radius: 20px; padding: 2px; width: 100%; }
  .toggle-group label { flex: 1; text-align: center; cursor: pointer; font-size: 0.75rem; font-weight: bold; }
  .toggle-group input { display: none; }
  .toggle-group span { display: block; padding: 4px 2px; border-radius: 20px; color: #ccc; }
  .orange input:checked + span { background: #f59e0b; color: #000; }
  .white input:checked + span { background: #fff; color: #000; }
  .narrow { max-width: 150px; margin-bottom: 16px; }

  .tile-display { display: flex; gap: 4px; margin: 20px 0; }
  .tile {
    background: #fff; color: #000; width: 52px; height: 68px; border-radius: 6px;
    display: flex; align-items: center; justify-content: center; font-size: 1.25rem; font-weight: bold;
    border-top: 6px solid #f59e0b; box-shadow: 0 2px 4px rgba(0,0,0,0.3);
  }

  .info-footer { text-align: center; font-family: monospace; }
  .fu-display { color: #aaa; margin-bottom: 2px; }
  .wait-btn { background: #333; color: #888; border: none; padding: 2px 8px; font-size: 0.7rem; border-radius: 4px; cursor: pointer; }
  .wait-btn.waiting { background: #0284c7; color: #fff; }

  .results { font-family: monospace; color: #aaa; }
  .final-score { color: #fff; font-size: 1.1rem; margin: 4px 0; }
  .reset-btn { background: #333; color: #ccc; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; margin-top: 8px; }
</style>
