<script>
  import { onDestroy, onMount } from "svelte";
  let startTime;
  let duration = 60;
  const UPDATE_FREQUENCY = 2000;
  const DURATION = 500;

  let left = duration;
  let int;
  let running = false;
  let startingText = "Click to start";

  onMount(() => updateText(startingText));
  onDestroy(() => clearInterval(int));
  async function start() {
    try {
      document.documentElement.requestFullscreen();
    } catch (e) {}
    duration =
      parseInt(new URLSearchParams(location.search).get("s")) || duration || 60;
    customText =
      "Don't move your mouse or leave this window. Just focus for " +
      formatSeconds(duration) +
      ".";
    await new Promise((r) => setTimeout(r, 3000));
    customText = null;
    startTime = Date.now();
    left = duration;
    running = true;
    int = setInterval(() => {
      let elapsed = (Date.now() - startTime) / 1000;
      left = duration - elapsed;
      if (left <= 0) {
        clearInterval(int);
        done();
      }
    }, UPDATE_FREQUENCY);
  }
  function resetVars() {
    if (int) {
      clearInterval(int);
    }
    startTime = null;
    running = false;
    left = duration;
    int = null;
  }
  async function done() {
    resetVars();
    try {
      document.exitFullscreen();
    } catch (e) {}
    customText = "Congratulations! You did it!";
    await new Promise((r) => setTimeout(r, 4000));
    customText = null;
  }
  let customText = null;
  $: text =
    customText ||
    (running ? Math.round(left) + " seconds left." : startingText);
  $: updateText(text);

  let textEl;
  let oldTextEl;
  async function updateText(text) {
    if (!(textEl && oldTextEl)) {
      return;
    }
    let oldText = textEl.innerText;
    textEl.innerText = text;
    oldTextEl.innerText = oldText;
    let opacity = 0;
    let start = Date.now();
    const setOpacity = (o) => {
      if (!(oldTextEl && textEl)) {
        return console.log({ oldTextEl, textEl });
      }
      oldTextEl.style.opacity = 1 - o;
      textEl.style.opacity = o;
    };
    while (Date.now() - start < DURATION) {
      opacity = (Date.now() - start) / DURATION;
      setOpacity(opacity);
      await new Promise((r) => setTimeout(r, 10));
    }
    setOpacity(1);
  }
  function formatSeconds(seconds) {
    var levels = [
      [Math.floor(seconds / 31536000), "years"],
      [Math.floor((seconds % 31536000) / 86400), "days"],
      [Math.floor(((seconds % 31536000) % 86400) / 3600), "hours"],
      [Math.floor((((seconds % 31536000) % 86400) % 3600) / 60), "minutes"],
      [(((seconds % 31536000) % 86400) % 3600) % 60, "seconds"],
    ];
    var returntext = "";

    for (var i = 0, max = levels.length; i < max; i++) {
      if (levels[i][0] === 0) continue;
      returntext +=
        " " +
        levels[i][0] +
        " " +
        (levels[i][0] === 1
          ? levels[i][1].substr(0, levels[i][1].length - 1)
          : levels[i][1]);
    }
    return returntext.trim();
  }
  function reset(reason) {
    return async () => {
      if (!running) {
        return;
      }
      if (duration - left < 4) {
        return;
      }
      resetVars();
      customText = `You ${reason}. It's ok.`;
      await new Promise((r) => setTimeout(r, 1000));
      customText = null;
    };
  }
</script>

<svelte:window
  on:click="{start}"
  on:mousemove="{reset('moved your mouse')}"
  on:mousedown="{reset('clicked')}"
  on:blur="{reset('left the page')}"
/>
<div class="container">
  <div class="old_text text" bind:this="{oldTextEl}">test</div>
  <span class="text" bind:this="{textEl}"></span>
</div>

<style>
  .container {
    width: 100vw;
    height: 100vh;
    background: #f2f2f2;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
      Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif;
    position: relative;
  }
  .text {
    top: 50%;
    left: 50%;
    position: absolute;
    transform: translate(-50%, -50%);
    font-size: 1.3em;
    font-weight: 100;
    text-align: center;
  }
</style>
