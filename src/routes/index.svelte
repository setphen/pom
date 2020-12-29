<script>
  import { onMount } from "svelte"

  const pomDuration = 25 * 60 * 1000    // 25 min
  const breakDuration = 5 * 60 * 1000   //  5 min

  let duration = pomDuration
  let running = false
  let timestamp = new Date()
  let now = new Date()
  let onBreak = false
  let appHidden = true

  // Start the "now" clock
  setInterval(() => {
    if (!running) return
    now = new Date()
  }, 50)

  $: elapsed = now - timestamp

  $: currentTotalDuration = onBreak ? breakDuration : pomDuration

  $: timeRemaining = Math.max(0, duration - elapsed)

  $: if (timeRemaining <= 0 && running == true) {
    console.log("DONE")
    reset()
    serialize()
  }

  function toggleBreak() {
    onBreak = !onBreak
    reset()
  }

  function reset(e) {
    duration = (onBreak ? breakDuration : pomDuration)
    running = false
    timestamp = new Date()
    now = new Date()
    serialize()
  }

  function startOrStop() {
    if (!running) {
      //Start new timer
      console.log("Starting")
      now = new Date()
      timestamp = new Date()
      running = true
      serialize()
    } else {
      // Pause
      console.log("Pausing")
      now = new Date()
      timestamp = new Date()
      duration = getRemainingTime()
      running = false
      serialize()
    }
  }

  function getRemainingTime() {
    return timeRemaining
  }

  onMount(() => {
    // deserialize from localstorage if available
    if (localStorage.getItem("running") !== null) {
      deserialize()

      if (!running) {
        timestamp = new Date()
        now = new Date()
      }

      appHidden = false
    }
  })

  function millisToMinutesAndSeconds(millis) {
    let ms = Math.ceil(millis/1000) * 1000 // round up
    let d = new Date(ms)
    let minutes = (d.getUTCMinutes() < 10 ? "0" : "") + d.getUTCMinutes()
    let seconds = (d.getUTCSeconds() < 10 ? "0" : "") + d.getUTCSeconds()
    return minutes + ':' + seconds
  }

  function serialize() {
    localStorage.setItem("running", running)
    localStorage.setItem("timestamp", timestamp.toJSON())
    localStorage.setItem("duration", duration.toString())
    localStorage.setItem("onBreak", onBreak)
  }

  function deserialize() {
    running = JSON.parse(localStorage.getItem("running")) // works for Boolean for some reason
    timestamp = new Date(Date.parse(localStorage.getItem("timestamp")))
    duration = parseInt(localStorage.getItem("duration"))
    onBreak = JSON.parse(localStorage.getItem("onBreak")) // " "
  }
</script>

<main>
  <button class="toggle" on:click={toggleBreak}>
    <span class="toggle__pom" class:active={!onBreak}>Pom</span>
    <span class="toggle__break" class:active={onBreak}>Break</span>
  </button>

  <div class="clock">
    <span class="time">
      {millisToMinutesAndSeconds(timeRemaining)}
    </span>
    <svg viewbox="0,0,200,200">
      <circle class="clock-path" cx=100 cy=100 r=98 stroke="#000" fill=none stroke-width=4
      transform="rotate(-90 100 100)" stroke-dasharray="616"
                                      stroke-dashoffset="{616 - 616 * (timeRemaining/currentTotalDuration)}"></circle>
      <circle class="timer-path" cx=100 cy=100 r=98 stroke="#000" fill=none stroke-width=4
      transform="translate(100 100) rotate(-90) scale(1 -1)
      translate(-100 -100)" stroke-dasharray="616"
                                      stroke-dashoffset="{616 * (timeRemaining/currentTotalDuration)}"></circle>
    </svg>
  </div>

  <div class="button-container">
    <button class="start-button" on:click={startOrStop}>
      <svg width="30" height="30" viewBox="0 0 30 30" fill="none" xmlns="http://www.w3.org/2000/svg">
        <g class:visible={!running}>
        <path d="M30 15L5 30L5 0L30 15Z" />
        </g>
        <g class:visible={running}>
        <path d="M1 0H10V30H1V0Z" />
        <path d="M20 0H29V30H20V0Z" />
        </g>
      </svg>
    </button>

    <button class="reset-button" on:click={reset}>Reset</button>
  </div>
</main>

<style>
  .clock svg {
    user-select: none;
    pointer-events: none;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  }

  main {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    width: 16rem;
  }

  button {
    appearance: none;
    border: 0;
    border-radius: 6px;
    background: transparent;
    cursor: pointer;
    color: var(--action);
  }

  button::-moz-focus-inner {
    border: 0;
  }

  .clock {
    width: 12rem;
    height: 12rem;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
  }

  .time {
    font-family: "Fira Code", monospace;
    font-size: 2rem;
    font-weight: 300;
    user-select: none;
  }

  .start-button,
  .reset-button {
    overflow: hidden;
  }

  .start-button {
    background-color: var(--action);
    color: var(--bg);
    height: 6rem;
    width: 6rem;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    position: relative;
    transition: background 0.1s;
    margin: 0;
  }

  .button-container {
    display: flex;
    justify-content: center;
    width: 100%;
    position: relative;
    margin-top: 2rem;
  }

  .start-button g {
    visibility: hidden;
    fill: var(--bg);
  }

  .start-button g.visible {
    visibility: visible;
    fill: var(--bg);
  }

  button:focus {
    box-shadow: 0 0 0 2px var(--bg), 0 0 0 4px var(--action);
    outline: 0;
  }

  .reset-button {
    align-items: center;
    background-color: var(--action);
    border-radius: 50%;
    color: var(--bg);
    display: flex;
    height: 4rem;
    justify-content: center;
    margin-left: -2rem;
    text-align: center;
    transition: background 0.1s;
    width: 4rem;
    position: absolute;
    left: 3rem;
    top: -2rem;
  }

  .clock-path {
    stroke: var(--action);
  }

  .timer-path {
    stroke: var(--accent);
  }

  .toggle {
    height: 4rem;
    width: 8rem;
    background-color: var(--accent);
    border-radius: 9rem;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0;
    margin-bottom: 2rem;
  }

  .toggle span {
    border-radius: 9rem;
    height: 4rem;
    width: 4rem;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 1rem;
    box-sizing: border-box;
  }

  .toggle .active {
    background-color: var(--action);
    color: var(--bg);
  }

  .toggle:hover .active,
  .start-button:hover,
  .reset-button:hover {
    background-color: var(--hover);
  }

  .appHidden {
    visibility: hidden;
  }
</style>
