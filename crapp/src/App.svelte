<script>

  // --- App State ---

  // 4 Meld Groups
  let melds = [
    { id: 1, type: 'PON', tileType: 'SIMPLE', openness: 'CLOSED', tiles: 3, wait: false },
    { id: 2, type: 'PON', tileType: 'HONOR',  openness: 'CLOSED', tiles: 3, wait: false },
    { id: 3, type: 'CHI', tileType: 'SIMPLE', openness: 'OPEN',   tiles: 3, wait: false },
    { id: 4, type: 'KAN', tileType: 'HONOR',  openness: 'OPEN',   tiles: 4, wait: false }
  ];

  // Janto (Pair / Eyes)
  let pairType = 'DRAGON'; // 'DRAGON', 'WIND', or 'NONE'
  let isWinningWaitOnPair = false; // Tanki wait adds 2 fu

  // General winning configs
  let winType = 'RON'; // 'RON' or 'TSUMO'
  let isMenzenRon = false; // Closed hand Ron adds 10 fu (handled in general hand configurations)

  // --- Dynamic UI Visual Sync ---
  // Automatically updates tile count based on whether a meld is transformed into a KAN
  $: {
    melds = melds.map(m => {
      m.tiles = m.type === 'KAN' ? 4 : 3;
      return m;
    });
  }

  // --- Reactive Fu Calculation Logic ---
  $: baseFu = 20;
  function calculateMeldFu(type,honor,o){
    if (type === 'CHI') return 0;
    let fu = type === 'KAN' ? 8 : 2;
    if ( honor === "HONOR" ) fu *= 2;
    if ( o === "CLOSED" ) fu *= 2;
    return fu;
  };
  $: meldsTotalFu = melds.reduce((sum, m) => {
    return sum + calculateMeldFu(m.type, m.tileType, m.openness);
  }, 0);

  $: pairFu = pairType === 'NONE' ? 0 : 2;

  // Wait fu checks (checks if any specific single meld or pair contains the winning tile wait)
  $: activeWaitMeld = melds.find(m => m.wait);
  $: waitFu = (activeWaitMeld && activeWaitMeld.type === 'CHI') || isWinningWaitOnPair ? 2 : 0;

  // Win condition modifiers
  $: winConditionFu = winType === 'TSUMO' ? 2 : (isClosedHand && winType === 'RON' ? 10 : 0);

  // Check if hand is fully concealed (all melds closed)
  $: isClosedHand = melds.every(m => m.openness === 'CLOSED');

  // Sum raw values
  $: totalRawFu = baseFu + meldsTotalFu + pairFu + waitFu + winConditionFu;

  // Round up to nearest 10 (Special case exception: 20 fu Tsumo pinned to 22 in some variants can be adjusted)
  $: roundedFu = totalRawFu === 20 ? 20 : Math.ceil(totalRawFu / 10) * 10;

  // Reset function
  function resetCalculator() {
    melds = melds.map(m => ({ ...m, type: 'CHI', tileType: 'SIMPLE', openness: 'OPEN', wait: false }));
    pairType = 'NONE';
    isWinningWaitOnPair = false;
    winType = 'RON';
  }

  // Toggle utility helper for wait selections (Only one group can hold the winning wait)
  function toggleWait(selectedId) {
    isWinningWaitOnPair = false;
    melds = melds.map(m => ({ ...m, wait: m.id === selectedId ? !m.wait : false }));
  }

  function togglePairWait() {
    melds = melds.map(m => ({ ...m, wait: false }));
    isWinningWaitOnPair = !isWinningWaitOnPair;
  }
</script>

<main class="dashboard">
  <!-- Grid Matrix of the 4 Melds -->
  <div class="melds-container">
    {#each melds as meld (meld.id)}
      <div class="meld-card">
        <!-- Selector Row 1: Chi / Pon / Kan -->
        <div class="btn-group orange-indicators">
          <button class:active={meld.type === 'CHI'} on:click={() => meld.type = 'CHI'}>CHI</button>
          <button class:active={meld.type === 'PON'} on:click={() => meld.type = 'PON'}>PON</button>
          <button class:active={meld.type === 'KAN'} on:click={() => meld.type = 'KAN'}>KAN</button>
        </div>

        <!-- Selector Row 2: Simple / Honor -->
        <div class="btn-group rounded-pills">
          <button class:active={meld.tileType === 'SIMPLE'} on:click={() => meld.tileType = 'SIMPLE'}>SIMPLE</button>
          <button class:active={meld.tileType === 'HONOR'} on:click={() => meld.tileType = 'HONOR'}>HONOR</button>
        </div>

        <!-- Visual Dynamic Tiles -->
        <div class="tiles-row">
          {#each Array(meld.tiles) as _, i}
            <div class="tile-block" class:highlighted={meld.type === 'CHI'}>
              {meld.tileType === 'SIMPLE' ? '#' : '☆'}
            </div>
          {/each}
        </div>

        <!-- Open / Closed Toggle -->
        <div class="btn-group segmented-capsule">
          <button class:active={meld.openness === 'OPEN'} on:click={() => meld.openness = 'OPEN'}>OPEN</button>
          <button class:active={meld.openness === 'CLOSED'} on:click={() => meld.openness = 'CLOSED'}>CLOSED</button>
        </div>

        <!-- Calculated Meld Individual Value Info Display -->
        <div class="info-block">
          <span class="value">[{meld.type === 'CHI' ? 0 : calculateMeldFu(meld.type, meld.tileType, meld.openness)}]</span>
          <button class="wait-btn" class:active={meld.wait} on:click={() => toggleWait(meld.id)}>WAIT</button>
        </div>
      </div>
    {/each}

    <!-- Janto Pair Block Component -->
    <div class="meld-card pair-card">
      <div class="btn-group rounded-pills">
        <button class:active={pairType === 'DRAGON'} on:click={() => pairType = 'DRAGON'}>DRAGON</button>
        <button class:active={pairType === 'WIND'} on:click={() => pairType = 'WIND'}>WIND</button>
        <button class:active={pairType === 'NONE'} on:click={() => pairType = 'NONE'}>PLAIN</button>
      </div>

      <div class="tiles-row">
        <div class="tile-block">#</div>
        <div class="tile-block">#</div>
      </div>

      <div class="info-block">
        <span class="value">[{pairFu}]</span>
        <button class="wait-btn" class:active={isWinningWaitOnPair} on:click={togglePairWait}>WAIT</button>
      </div>
    </div>
  </div>

  <!-- Bottom Win Condition Panel -->
  <div class="footer-controls">
    <div class="btn-group overall-win-toggle">
      <button class:active={winType === 'RON'} on:click={() => winType = 'RON'}>RON</button>
      <button class:active={winType === 'TSUMO'} on:click={() => winType = 'TSUMO'}>TSUMO</button>
    </div>

    <!-- Live Running Score Terminal Tracker -->
    <div class="score-readout">
      <p class="base-note">base: [{baseFu}]</p>
      <h3>Fu Score is {totalRawFu} -> <span class="rounded-target">({roundedFu})</span></h3>
      <button class="reset-action" on:click={resetCalculator}>RESET</button>
    </div>
  </div>
</main>

<style>
  /* Styling to capture your dark UI palette framework */
  .dashboard {
    background-color: #121212;
    color: #ffffff;
    padding: 24px;
    font-family: monospace;
  }
  .melds-container {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
    gap: 16px;
    margin-bottom: 30px;
  }
  .meld-card {
    background-color: #1c1c1e;
    padding: 16px;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 12px;
  }
  .btn-group {
    display: flex;
    width: 100%;
    background-color: #2c2c2e;
    border-radius: 4px;
    overflow: hidden;
  }
  .btn-group button {
    flex: 1;
    background: transparent;
    border: none;
    color: #8e8e93;
    padding: 6px 2px;
    font-size: 0.75rem;
    font-weight: bold;
    cursor: pointer;
  }
  /* Specific styling modes mimicking your visual indicators */
  .orange-indicators button.active {
    background-color: #ff9f0a;
    color: #000000;
  }
  .rounded-pills { border-radius: 12px; }
  .rounded-pills button.active {
    background-color: #ffffff;
    color: #000000;
  }
  .segmented-capsule button.active {
    background-color: #ff9f0a;
    border-radius: 12px;
    color: #000000;
  }
  .tiles-row {
    display: flex;
    gap: 4px;
    margin: 8px 0;
  }
  .tile-block {
    width: 32px;
    height: 45px;
    background: #ffffff;
    color: #000000;
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.2rem;
    font-weight: bold;
    border-top: 4px solid #ff9f0a;
  }
  .tile-block.highlighted {
    border-top: 4px solid #ff9f0a;
  }
  .info-block {
    text-align: center;
    color: #8e8e93;
  }
  .wait-btn {
    display: block;
    margin-top: 4px;
    background: #2c2c2e;
    border: none;
    color: #666;
    padding: 2px 8px;
    font-size: 0.7rem;
    cursor: pointer;
  }
  .wait-btn.active {
    color: #ff9f0a;
    background: #3a3a3c;
  }
  .footer-controls {
    text-align: center;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
  }
  .overall-win-toggle {
    width: 200px;
    border-radius: 20px;
  }
  .overall-win-toggle button.active {
    background-color: #ff9f0a;
    color: #000000;
  }
  .rounded-target {
    color: #ff9f0a;
    font-size: 1.6rem;
  }
  .reset-action {
    background: #2c2c2e;
    color: #ffffff;
    border: 1px solid #444;
    padding: 6px 16px;
    cursor: pointer;
  }
</style>
