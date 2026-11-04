<svelte:options runes={true} /* Don't accept legacy syntax */ />

<script>
  // initialize hand state
  const defaultMelds = new Array(4).fill({ type: "CHI", isSimple: true, isOpen: false, isWait: false });
  let melds = $state([ ...defaultMelds ]);

  let meldFu = $derived(melds.map((m) => calculateMeldFu(m.type, m.isSimple, m.isOpen)));

  // separate logic for pair value
  const defaultPair = {Dragon: false, Wind: false, Seat: false, Round: false};
  let pair = $state({ ...defaultPair });
  let pairFu = $derived((pair.Dragon ? 2:0) + (pair.Wind && pair.Seat ? 2:0) + (pair.Wind && pair.Round ? 2:0));
  let pairSymbol = $derived({0:"#", 2:"☆", 4:"★"}[pairFu]);
  let pairIsWait = $state(false);

  let isTsumo = $state(false);
  let base = $derived(20 + (isTsumo? 2:0) );

  let totalFu = $derived(base + pairFu + meldFu.reduce((a,b)=>a+b,0));

  function calculateMeldFu(type, simple, open) {
    if (type === "CHI") return 0;
    let base = type === "KAN" ? 8 : 2; // PON is 2, KAN is 8
    if (!simple) base *= 2;
    if (!open) base *= 2; // Double for closed sets
    return base;
  }
  function reset() {
    melds = [ ...defaultMelds ];
    pair = { ...defaultPair };
    isTsumo = false;
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
        <!-- Tile Group Visualizer -->
        <div class="tile-display" class:open={meld.isOpen} class:chi={meld.type==="CHI"}>
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

        <!-- Meld Type Picker -->
        <div class="toggle-group orange">
          <label><input type="radio" bind:group={meld.type} value="CHI" onclick={() => meld.isSimple = true}/><span>CHI</span></label>
          <label><input type="radio" bind:group={meld.type} value="PON" /><span>PON</span></label>
          <label><input type="radio" bind:group={meld.type} value="KAN" /><span>KAN</span></label>
        </div>

        <!-- Tile value picker -->
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

        <!-- intermediate Fu value -->
        <div class="info-footer">
          <div class="lcd">{meldFu[index]}</div>
          <button class="wait-btn" class:waiting={meld.isWait} onclick={() => meld.isWait = !meld.isWait}>
            WAIT
          </button>
        </div>
      </div>
    {/each}

    <!-- ...and the pair -->
    <div class="card" class:active-wait={pairIsWait}>
      <div class="tile-display">
        <div class="tile">{pairSymbol}</div>
        <div class="tile">{pairSymbol}</div>
      </div>
      <div class="toggle-group white">
        <label><input type="checkbox" bind:checked={pair.Dragon} onclick={()=> pair.Wind=false}/><span>DRAGON</span></label>
        <label><input type="checkbox" bind:checked={pair.Wind}  onclick={()=> pair.Dragon=false}/><span>WIND</span></label>
      </div>
      <div class="toggle-group white" class:disabled={!pair.Wind}>
        <label><input type="checkbox" bind:checked={pair.Round}/><span>ROUND</span></label>
        <label><input type="checkbox" bind:checked={pair.Seat} /><span>SEAT</span></label>
      </div>

      <div class="toggle-group white disabled"><span>#</span></div>


      <div class="info-footer">
        <div class="lcd">{pairFu}</div>
        <button class="wait-btn" class:waiting={pairIsWait} onclick={() => pairIsWait = !pairIsWait}>
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
      <p>base: [{totalFu}]</p>
      <p class="final-score">Score is {totalFu} -> ({Math.ceil(totalFu / 10.0) * 10 })</p>
      <button class="reset-btn" onclick={reset}>RESET</button>
    </div>
  </footer>
</main>

<style>
  @font-face {
    font-family: 'Ubuntu Embedded';
    src: url('fonts/Ubuntu-Light.ttf') format('truetype');
    font-style: normal;
  }
  @font-face {
    font-family: 'LCD Segment';
    src: url('fonts/LCDAT_TPhoneTimeDate.ttf') format('truetype');
  }

  :global(body) { background-color: #121212; color: #fff; margin: 0; }
  .app-container { max-width: 1800px; margin: 0 auto; padding: 16px; font-family: sans-serif; }

  h1 { font-size: 1.2rem; margin: 0; text-align: center; color: #eee; }
  .subtitle { font-size: 0.9rem; color: #777; margin: 2px 0 16px 0; }

  .board-layout {
    display: flex;
    flex-direction: row;       /* Force horizontal alignment by default */
    flex-wrap: nowrap;         /* Keep entire hand on one line */
    justify-content: center;   /* Center items together, horizontally on the screen */
    gap: 16px;                 /* Spacing between the card groups */
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
    width: 90%;
  }
  .toggle-group label {
    flex: 1;
    text-align: center;
    cursor: pointer;
    font-size: 0.75rem;
    font-weight: bold;
  }
  .toggle-group span {
    display: block;
    padding: 4px 2px;
    border-radius: 20px;
    color: #ccc;
  }
  .toggle-group input { display: none; }
  .orange input:checked + span { background: #fb9c01; color: #000; }
  .white input:checked + span { background: #fff; color: #000; }
  .narrow { max-width: 150px; margin-bottom: 16px; }

  .disabled { opacity: 0.0; pointer-events: none; }

  /* Tile Visual Assets */
  .tile-display { display: flex; gap: 2px; margin: 4px 0; min-height: 70px; }
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
    border-top: 8px solid #fb9c01; /* Yellow tile back peeking out, implying oblique perspective */
    box-shadow: 0 2px 4px rgba(0,0,0,0.3);
  }
  /* oblique perspective, implies tile is face-up on table */
  .open .tile { border-bottom: 8px solid #fb9c01; border-top: none; }
  /* increasing font size for sequences */
  .chi .tile:first-child { font-size: 1.5rem; }
  .chi .tile             { font-size: 2rem; }
  .chi .tile:last-child  { font-size: 2.5rem; }

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
