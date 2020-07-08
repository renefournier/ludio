<script>
  import { onMount, onDestroy, createEventDispatcher } from "svelte";
  import SortableList from "svelte-sortable-list";
  import PlayButton from "./components/playButton.svelte";

  import { Container, Col, Row, Button } from "sveltestrap";
  import MarkerButton from "./components/markerButton.svelte";
  import Render from "./components/render.svelte";
  import Clip from "./components/clip.svelte";
  import Item from "./components/item.svelte";
  import videos from "./data/videos.json";
  import sampleClips from "./data/clips.json";

  console.log(sampleClips);

  window.playerSource = {}; // Attach to window so I can monkey with the player instance from DevTools
  window.playerPreview = {};

  let playerActive = "source",
    sourceMedia,
    currentTime,
    start = null,
    stop = null,
    elapsed = null,
    clips = [];

  const handleKeydown = event => {
    const keyCode = event.keyCode;
    console.log(keyCode);
    if ([32, 73, 69, 37, 39, 40, 38, 69, 77].indexOf(keyCode) > -1) {
      event.preventDefault();
    } else {
      return;
    }
    switch (
      keyCode // mark in/out, scrub
    ) {
      case 32: // pause/play
        break;
      case 73: // i == trim start
        markIn();
        break;
      case 69: // o == trim stop
        markOut();
        break;
      case 37: // left arrow rwd one frame
        scrub("←");
        break;
      case 39: // right arrow fwd one frame
        scrub("→");
        break;
      case 40: // go to beginning
        scrub("⇤");
        break;
      case 38: // go to to end
        scrub("⇥");
        break;
      case 77: // mute/unmute
        muteToggle();
        break;
      case 69: // export
        console.log("Export!");
        break;
    }
  };

  const muteToggle = () => {
    switch (playerActive) {
      case "source":
        playerSource.muted(!playerSource.muted());
        break;
      case "preview":
        playerPreview.muted(!playerPreview.muted());
        break;
    }
  };

  const scrub = direction => {
    playerSource.pause(); // hold your horses, peoples
    let s, duration, newTime;
    switch (playerActive) {
      case "source":
        s = playerSource.currentTime();
        duration = playerSource.duration();
        break;
      case "preview":
        s = playerPreview.currentTime();
        duration = playerPreview.duration();
        break;
    }
    switch (direction) {
      case "←":
        newTime = s - 1 / 30; // one frame?
        break;
      case "→":
        newTime = s + 1 / 30; // one frame?
        break;
      case "⇥":
        newTime = duration;
        break;
      case "⇤":
        newTime = 0;
        break;
    }
    playerSource.currentTime(newTime);
  };

  const markIn = () => {
    const newTime = parseFloat(playerSource.currentTime().toFixed(3));
    if (start || stop) {
      stop = null;
    }
    start = newTime;
  };

  const markOut = () => {
    const newTime = parseFloat(playerSource.currentTime().toFixed(3));
    if (!start) {
      // do nothing
    } else {
      stop = newTime;
      clips = [
        ...clips,
        { id: Date.now(), start: start, stop: stop, duration: stop - start }
      ];
      stop = null;
      start = null;
    }
  };

  const plurality = (number, singular, plural) => {
    if (number === 1) return singular;
    return plural;
  };

  let clipsDuration = "";

  $: if (clips.length > 0) {
    let clipsDurationSeconds = 0;
    clips.forEach(clip => {
      clipsDurationSeconds += clip.duration;
    });
    clipsDuration = new Date(clipsDurationSeconds * 1000)
      .toISOString()
      .substr(11, 10);
  }

  const sortList = ev => {
    clips = ev.detail;
  };

  const play = async media => {
    if (typeof media === "object") {
      // Play 'em all — to do
      media.forEach(item => {
        console.log(item);
      });
    } else {
      // Just one
      // console.log(`Playing ${media}`);
      const src = clips[media].src;
      const start = clips[media].start;
      await playerPreview.src(src);
      await playerPreview.currentTime(start);
      await playerPreview.play();
    }
  };

  const del = media => {
    if (typeof media === "object") {
      clips = [];
    } else {
      if (media > -1) {
        console.log(`Deleting ${media}`);
        const deletedClip = clips.splice(media, 1);
        clips = [...clips];
      }
    }
  };

  const render = () => {
    console.log("Render!");
  };

  onMount(async () => {
    console.log("M: 🏇");

    sourceMedia = videos[Math.floor(Math.random() * videos.length)]; // let’s pick one at random, shall we? It’s all great content, so let’s not be too picky, right?!

    clips = sampleClips.map(clip => {
      // add random sample sourcemedia
      clip.src = videos[Math.floor(Math.random() * videos.length)];
      return clip;
    });

    window.playerSource = videojs(
      "video-source",
      {
        autoplay: true,
        controls: true,
        sources: sourceMedia
      },
      function onPlayerReady() {
        console.log("onPlayerReady0");
        playerSource.muted(true);
      }
    );

    playerSource.on("play", function(e) {
      playerActive = "source";
      playerPreview.pause();
    });

    window.playerPreview = videojs(
      "video-preview",
      {
        autoplay: false,
        controls: true,
        sources: videos[Math.floor(Math.random() * videos.length)] // pick a random video
      },
      function onPlayerReady() {
        console.log("onPlayerReady1");
        playerPreview.muted(true);
      }
    );

    playerPreview.on("play", function(e) {
      playerActive = "preview";
      playerSource.pause();
    });
  });

  onDestroy(async () => {
    console.log("U: 🐎 "); // rider falls off
  });

  setInterval(function() {
    const seconds = playerSource.currentTime();
    currentTime = new Date(seconds * 1000).toISOString().substr(11, 10);
    if (start) {
      elapsed = seconds - start;
    }
  }, 100);
</script>

<style>
  .monitor {
    background: rgba(0, 0, 0, 0.7);
  }

  .monitor > p {
    text-align: center;
    display: block;
    color: rgba(255, 255, 255, 0.1);
    width: 100%;
  }

  .monitor > p.active {
    color: #00b4ff;
    text-shadow: 0 1px 10px rgba(0, 0, 0, 0.25);
  }
  .video {
    max-width: 100%;
    max-height: 350px;
  }
  ul {
    width: 100% !important;
  }

  ul li {
    width: 100% !important;
  }
</style>

<svelte:window on:keydown={handleKeydown} />

<Container fluid class="px-0">
  <Row class="mx-0 px-0 w-100">
    <Col sm="6" class="mx-0 px-0">

      <Row class="m-0 p-0 monitor">
        <p class:active={playerActive === 'source'} class="px-4 py-2 mb-0 ">
          <span class="fa fa-sign-in-alt mr-1" />
          <strong>Source</strong>
        </p>
      </Row>

      <Row class="m-0 p-0">
        <video id="video-source" class="video video-js" />
      </Row>

      <Row class="mx-0 px-0">
        <Col sm="6" class="mx-0 px-0">
          <MarkerButton {currentTime} {start} {stop} {markIn} {markOut} />
        </Col>
        <Col sm="6" class="mx-0 px-0">
          <div class="p-2 text-center">
            <span class="muted">{currentTime}</span>
            <strong class="muted ml-3">
              {#if elapsed}{String(`${Number(elapsed).toFixed(1)}s`) || ''}{/if}
            </strong>
          </div>
        </Col>
      </Row>

      <Row class="mx-0 px-0">
        <Col sm="6" class="mx-0 px-0">

          <h5 class="mx-3 my-3">
            <span class="muted">{clipsDuration}s</span>
          </h5>
        </Col>
        <Col sm="6" class="mx-0 px-0 text-right">
          <h5 class="mx-3 my-3 float-right">
            <PlayButton
              size="lg"
              del={() => del(clips)}
              play={() => play(clips)} />
          </h5>
        </Col>
      </Row>

      <Row class="mx-0 px-0 w-100">

        <SortableList
          key="id"
          list={clips}
          on:sort={sortList}
          let:item
          let:index>
          <Item
            {item}
            {index}
            del={() => del(index)}
            play={() => play(index)} />
        </SortableList>

      </Row>

    </Col>

    <Col
      sm="6"
      class="m-0 p-0"
      style="border-left: 1px solid rgba(0,0,0,0.25);">

      <Row class="m-0 p-0 monitor">
        <p class:active={playerActive === 'preview'} class="px-4 py-2 mb-0 ">
          <span class="fa fa-tv mr-1" />
          <strong>Preview</strong>
        </p>
      </Row>

      <Row class="m-0 p-0">
        <video
          id="video-preview"
          class="video video-js vjs-big-play-centered" />
      </Row>

      <Row class="mx-0 px-0">
        <Col sm="6" class="mx-0 px-0">
          <Render {clips} {render} />
        </Col>
      </Row>

    </Col>
  </Row>

  <Row class="mx-0 px-0 w-100" style="border-left: 1px solid rgba(0,0,0,1)">
    <Col class="p-2">
      <code>{JSON.stringify(clips)}</code>
    </Col>

  </Row>
</Container>
