<script>
  // Expose variables so the parent app can read this meld's state
  export let type = "CHI";        // CHI, PON, KAN
  export let isSimple = true; // SIMPLE, HONOR/TERMINAL
  export let isOpen = false;      // false = CLOSED, true = OPEN
  export let isWait = false;      // Is this the winning wait tile?

  // Reactive Fu calculation directly tied to your visual states
  $: fuValue = calculateMeldFu(type, isSimple, isOpen);

  function calculateMeldFu(type, simple, open) {
    if (type === "CHI") return 0;
    let base = type === "KAN" ? 8 : 2; // PON is 2, KAN is 8
    if (!simple) base *= 2;
    if (!open) base *= 2; // Double for closed sets
    return base;
  }
</script>

<div class="meld-card" class:active-wait={isWait}>
  <!-- Top Segmented Control (Meld Type) -->
  <div class="toggle-group orange">
    <label><input type="radio" bind:group={type} value="CHI" /><span>CHI</span></label>
    <label><input type="radio" bind:group={type} value="PON" /><span>PON</span></label>
    <label><input type="radio" bind:group={type} value="KAN" /><span>KAN</span></label>
  </div>

  <!-- Middle Segmented Control (Tile Value Type) -->
  <div class="toggle-group white" class:disabled={type === "CHI"}>
    <label>
      <input type="radio" bind:group={isSimple
  } value={true} disabled={type === "CHI"} />
      <span>SIMPLE</span>
    </label>
    <label>
      <input type="radio" bind:group={isSimple
  } value={false} disabled={type === "CHI"} />
      <span>HONOR</span>
    </label>
  </div>

  <!-- Visual Tile Graphic Group -->
  <div class="tile-display">
    <div class="tile">{isSimple ? "#" : "☆"}</div>
    <div class="tile">{isSimple ? "#" : "☆"}</div>
    <div class="tile">{isSimple ? "#" : "☆"}</div>
    {#if type === "KAN"}
      <div class="tile">{isSimple ? "#" : "☆"}</div>
    {/if}
  </div>

  <!-- Open/Closed Control -->
  <div class="toggle-group orange">
    <label><input type="radio" bind:group={isOpen} value={true} /><span>OPEN</span></label>
    <label><input type="radio" bind:group={isOpen} value={false} /><span>CLOSED</span></label>
  </div>

  <!-- Fu Status Details -->
  <div class="info-footer">
    <div class="fu-display">[{fuValue}]</div>
    <button class="wait-btn" class:waiting={isWait} on:click={() => isWait = !isWait}>
      WAIT
    </button>
  </div>
</div>

<style>
  .meld-card {
background: #1e1e1e;
    border: 2px solid transparent;
    padding: 3px;
    border-radius: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
    flex: 1 1 140px;          /* Matches the pair card behavior */
    max-width: 180px;
  }
  .active-wait { border-color: #0284c7; }

  /* Segmented Capsule Toggles */
  .toggle-group {
    display: flex;
    background: #333;
    border-radius: 20px;
    overflow: hidden;
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
  .toggle-group input { display: none; }
  .toggle-group span {
    display: block;
    padding: 4px 2px;
    border-radius: 20px;
    color: #ccc;
  }

  /* Orange Theme Active State */
  .orange input:checked + span { background: #fb9c01; color: #000; }
  /* White Theme Active State */
  .white input:checked + span { background: #fff; color: #000; }
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
    font-family: monospace;
    /* font-weight: bold; */
    border-top: 6px solid #fb9c01; /* Orange top border accents matching your app */
    /* box-shadow: 0 2px 4px rgba(0,0,0,0.3); */
  }

  .info-footer { text-align: center; color: #fff; font-family: monospace; }
  .fu-display { margin-bottom: 2px; color: #aaa; }

  .wait-btn {
    background: #333; color: #888; border: none; padding: 2px 8px;
    font-size: 0.7rem; border-radius: 4px; cursor: pointer;
  }
  .wait-btn.waiting { background: #0284c7; color: #fff; }
</style>
