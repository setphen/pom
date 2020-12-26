<script>
  import { onMount } from "svelte"

  const pomDuration = 25 * 60 * 1000    // 25 min
  const breakDuration = 5 * 60 * 1000   //  5 min

  let duration = pomDuration
  let running = false
  let timestamp = new Date()
  let now = new Date()
  let onBreak = false

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

    //Switch to break/pom
    onBreak = !onBreak
    reset()
    serialize()
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
    }
  })

  function millisToMinutesAndSeconds(millis) {
    let ms = Math.ceil(millis/1000) * 1000 // round up
    let d = new Date(ms)
    let minutes = d.getUTCMinutes()
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
  <button class="start-button" on:click={startOrStop}>
    <span>
      {running ? "Pause" : "Start"} {onBreak ? "Break" : "Pom"}
    </span>

    <span class="time">
      {millisToMinutesAndSeconds(timeRemaining)}
    </span>
  </button>


  <svg viewbox="0,0,200,200">
    <circle class="clock-path" cx=100 cy=100 r=98 stroke="#000" fill=none stroke-width=2
    transform="rotate(-90 100 100)" stroke-dasharray="616"
                                    stroke-dashoffset="{616-616 *
                                    (timeRemaining/currentTotalDuration)}"></circle>
    <circle class="timer-path" cx=100 cy=100 r=98 stroke="#000" fill=none stroke-width=2
    transform="rotate(-90 100 100)" stroke-dasharray="616"
                                    stroke-dashoffset="{-616 *
                                    (timeRemaining/currentTotalDuration)}"></circle>
  </svg>
  <button class="reset-button" on:click={() => {onBreak = false; reset();}}>Reset</button>
  <!--
  <table>
    <tr>
      <th>running</th>
      <td>{running}</td>
    </tr>
    <tr>
      <th>duration</th>
      <td>{duration}</td>
    </tr>
    <tr>
      <th>remaining ms</th>
      <td>{timeRemaining}</td>
    </tr>
    <tr>
      <th>elapsed</th>
      <td>{elapsed}</td>
    </tr>
    <tr>
      <th>timestamp</th>
      <td>{timestamp.toJSON()}</td>
    </tr>
    <tr>
      <th>now</th>
      <td>{now.toJSON()}</td>
    </tr>
  </table>
  -->

</main>

<style>
  svg {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    user-select: none;
    pointer-events: none;
  }

  main {
    position: relative;
    height: 16rem;
    width: 16rem;
    display: grid;
    place-content: center;
  }

  button {
    appearance: none;
    border: 0;
    border-radius: 6px;
    background: transparent;
    cursor: pointer;
    color: #9C5400;
  }

  button::-moz-focus-inner {
    border: 0;
  }

  .time {
    font-family: monospace;
    font-size: 0.8rem;
    position: absolute;
    top: 118%;
  }

  .start-button {
    background: #FFC581;
    height: 8rem;
    width: 8rem;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    position: relative;
    transition: background 0.1s;
    margin: 0;
  }

  .start-button:active {
    background: #FAB564;
  }

  .reset-button:focus,
  .start-button:focus {
    box-shadow: 0 0 20px 4px white;
    outline: 0;
  }

  .reset-button {
    background: #FFC581;
    height: 4rem;
    width: 4rem;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    transition: background 0.1s;
    position: absolute;
    top: 112%;
    left: 50%;
    margin-left: -2rem;
    text-align: center;
  }

  .reset-button:hover {
    background: #FAB564;
  }

  .clock-path {
    stroke: #FF4D00;
  }
  .timer-path {
    stroke: #FFC581;
  }
</style>
