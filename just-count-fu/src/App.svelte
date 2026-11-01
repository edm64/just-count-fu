<script>
  // Instantiate hand state
  let melds = [
    { type: "CHI", isSimple: true, isOpen: false, isWait: false },
    { type: "CHI", isSimple: true, isOpen: false, isWait: false },
    { type: "CHI", isSimple: true, isOpen: false, isWait: false },
    { type: "CHI", isSimple: true, isOpen: false, isWait: false }
  ];

  // separate logic for pair value
  let pair = {dragon: false, wind: false, seat: false, round: false};
  let pairDragon = false;
  let pairWind = false;
  let pairSeat = false;
  let pairRound = false;
  let pairIsWait = false;

  let isTsumo = false;
  let base = 20;
  if (isTsumo) { base += 2};

  // Reactively tally up all your separate Fu calculations automatically
  $: totalFu = calculateTotalFu(melds, false, pairIsWait, isTsumo);

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
      <div class="card" class:active-wait={meld.isWait}>
        <!-- Visual Tile Graphic Group -->
        <div class="tile-display">
          <div class="tile">{meld.isSimple ? "#" : "☆"}</div>
          <div class="tile">{meld.isSimple ? "#" : "☆"}</div>
          <div class="tile">{meld.isSimple ? "#" : "☆"}</div>
          {#if meld.type === "KAN"}
          <div class="tile">{meld.isSimple ? "#" : "☆"}</div>
          {/if}
        </div>

        <!-- Open/Closed Control -->
        <div class="toggle-group orange">
          <label><input type="radio" bind:group={meld.isOpen} value={true } /><span> OPEN </span></label>
          <label><input type="radio" bind:group={meld.isOpen} value={false} /><span>CLOSED</span></label>
        </div>

        <!-- Top Segmented Control (Meld Type) -->
        <div class="toggle-group orange">
          <label><input type="radio" bind:group={meld.type} value="CHI" on:click={() => meld.isSimple = true}/><span>CHI</span></label>
          <label><input type="radio" bind:group={meld.type} value="PON" /><span>PON</span></label>
          <label><input type="radio" bind:group={meld.type} value="KAN" /><span>KAN</span></label>
        </div>

        <!-- Middle Segmented Control (Tile Value Type) -->
        <div class="toggle-group orange" class:disabled={meld.type === "CHI"}>
          <label>
            <input type="radio" bind:group={meld.isSimple} value={true}/>
            <span>SIMPLE</span>
          </label>
          <label>
            <input type="radio" bind:group={meld.isSimple} value={false}/>
            <span>HONOR</span>
          </label>
        </div>

        <!-- Fu Status Details -->
        <div class="info-footer">
          <div class="lcd">6</div>
          <button class="wait-btn" class:waiting={meld.isWait} on:click={() => meld.isWait = !meld.isWait}>
            WAIT
          </button>
        </div>
      </div>
    {/each}

    <!-- Simpler manual pair controller card styling -->
    <div class="card" class:active-wait={pairIsWait}>
      <div class="tile-display">
        <div class="tile">#</div>
        <div class="tile">#</div>
      </div>
      <div class="toggle-group white">
        <label><input type="checkbox" bind:checked={pairDragon} on:click={()=> pairWind=false}/><span>DRAGON</span></label>
        <label><input type="checkbox" bind:checked={pairWind}  on:click={()=> pairDragon=false}/><span>WIND</span></label>
      </div>
      <div class="toggle-group white" class:disabled={!pairWind}>
        <label><input type="checkbox" bind:checked={pairRound}/><span>ROUND</span></label>
        <label><input type="checkbox" bind:checked={pairSeat} /><span>SEAT</span></label>
      </div>


      <div class="info-footer">
        <div class="lcd">0</div>
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
  @font-face {
    font-family: 'Ubuntu Embedded';
    src: url('fonts/Ubuntu-Light.ttf') format('truetype');
    /*font-weight: 300;*/
    font-style: normal;
  }
  @font-face {
    font-family: 'LCD Segment';
    src: url('fonts/LCDAT_TPhoneTimeDate.ttf') format('truetype');
  }

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

  .card {
    border: 2px solid transparent;
    padding: 3px;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
    flex: 1 1 140px;
    max-width: 180px;
  }
  .active-wait { border-color: #0284c7; }

  /* Segmented option choosers */
  .toggle-group {
    display: flex;
    background: #333;
    border-radius: 20px;
    overflow:hidden;
    padding: 2px;
    width: 80%;
  }
  .toggle-group label {
    flex: 1;
    text-align: center;
    cursor: pointer;
    font-size: 0.85rem;
    font-weight: bold;
  }
  .toggle-group span {
    display: block;
    padding: 4px 2px;
    border-radius: 20px;
    color: #ccc;
  }
  .toggle-group input { display: none; }
  .orange input:checked + span { background: #f59e0b; color: #000; }
  .white input:checked + span { background: #fff; color: #000; }
  .narrow { max-width: 150px; margin-bottom: 16px; }

  .disabled { opacity: 0.0; pointer-events: none; }

  /* Tile Visual Assets */
  .tile-display { display: flex; gap: 4px; margin: 4px 0; min-height: 70px; }
  .tile {
    background: #fff;
    color: #000;
    width: 52px;
    height: 68px;
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2.25rem;
    /* font-weight: bold; */
    border-top: 6px solid #fb9c01; /* Orange top border accents matching your app */
    box-shadow: 0 2px 4px rgba(0,0,0,0.3);
  }

  .lcd {
    font-family: 'LCD Segment';
    font-size: 2rem;
    text-align: right;
    /* background:#b9e9ad; */
    background:hsl(80, 61%, 85%);
    color: #000;
    padding: 2px;
    padding-top: 6px;
  }
  .info-footer { text-align: center; font-family: monospace; }

  .wait-btn {
    background: #333; color: #888; border: none; padding: 2px 8px;
    font-size: 0.7rem; border-radius: 4px; cursor: pointer;
  }
  .wait-btn.waiting { background: #0284c7; color: #fff; }

  .results { font-family: monospace; color: #aaa; }
  .final-score { color: #fff; font-size: 1.1rem; margin: 4px 0; }
  .reset-btn { background: #333; color: #ccc; border: none; padding: 6px 12px; border-radius: 4px; cursor: pointer; margin-top: 8px; }
</style>
