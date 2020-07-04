<script>
  import { onMount, onDestroy, createEventDispatcher } from "svelte";
  // import { draggable } from "./lib/dragdrop.js";
  // import { crossfade } from "svelte/transition";
  // import { quintOut, elasticOut } from "svelte/easing";
  // import { flip } from "svelte/animate";
  import SortableList from "svelte-sortable-list";
  import PlayButton from "./components/playButton.svelte";

  import { Container, Col, Row, Button } from "sveltestrap";
  import MarkerButton from "./components/markerButton.svelte";
  import Clip from "./components/clip.svelte";
  import Item from "./components/item.svelte";
  import videos from "./data/videos.json";

  window.playerSource = {}; // Attach to window so I can monkey with the player instance from DevTools
  window.playerPreview = {};

  let sourceMedia, key, keyCode, currentTime;
  let start = null,
    stop = null;
  let clips = [{ id: Date.now(), start: 5.23, stop: 6.57, duration: 1.34 }];

  const handleKeydown = event => {
    key = event.key;
    keyCode = event.keyCode;
    if (keyCode === 73) {
      // i == trim start
      markIn();
    } else if (keyCode === 79) {
      // o == trim stop
      markOut();
    }
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

  onMount(async () => {
    console.log("M: 🏇");

    sourceMedia = videos[Math.floor(Math.random() * videos.length)]; // let’s pick one at random, shall we?

    window.playerSource = videojs(
      "video-source",
      {
        autoplay: true,
        controls: true,
        sources: sourceMedia
      },
      function onPlayerReady() {
        console.log("onPlayerReady0");
      }
    );

    window.playerPreview = videojs(
      "video-preview",
      {
        autoplay: false,
        controls: true,

        sources: videos[Math.floor(Math.random() * videos.length)] // pick a random video
      },
      function onPlayerReady() {
        console.log("onPlayerReady1");
      }
    );
  });

  onDestroy(async () => {
    console.log("U: 🐎 ");
  });

  let now = Date();
  setInterval(function() {
    now = Date();
    const seconds = playerSource.currentTime();
    currentTime = new Date(seconds * 1000).toISOString().substr(11, 10);
  }, 100);
</script>

<style>
  h1 {
    color: #ff3e00;
    /* text-transform: uppercase; */
    font-size: 4em;
    font-weight: 100;
    margin: 1rem;
  }
  h2 {
    text-align: center;
    font-weight: 300;
    font-size: 1.5rem;
    color: #999;
  }
  h3 {
    text-align: center;
    font-weight: 300;
    font-size: 1.25rem;
    color: #777;
  }
  .src {
    width: 100%;
    padding: 0;
    margin: 0;
    display: inline;
    overflow-x: scroll;
  }

  /* .video {
    max-width: 600px;
    max-height: 400px;
    border-radius: 10px;
    border: 1px solid rgba(0, 0, 0, 0.02);
    background: #222;
    box-shadow: 0 3px 15px -3px rgba(0, 0, 0, 1);
    margin: 1rem auto;
    style: 10px solid red;
  } */

  .video {
    max-width: 100%;
    max-height: 350px;
  }

  .box {
    /* width: 50vw; */
    /* max-height: 500px; */
    /* border: 1px solid red; */
  }
</style>

<svelte:window on:keydown={handleKeydown} />

<Container fluid class="px-0">
  <Row class="mx-0 px-0 w-100">
    <Col sm="6" class="mx-0 px-0">
      <div class="box">
        <video id="video-source" class="video video-js" />
      </div>
      <MarkerButton {currentTime} {start} {stop} {markIn} {markOut} />
      <Row class="mx-0 px-0 w-100">
        <Col sm="6" class="mx-0 px-0">

          <h3 class="my-2">
            <span class="muted">
              {clips.length} {plurality(clips.length, 'clip', 'clips')}
            </span>
            <strong class="ml-3" style="font-weight: 700">
              {clipsDuration}s
            </strong>
          </h3>
        </Col>
        <Col sm="5" class="mx-0 px-0">
          <h3 class="p-2 float-right">
            <PlayButton
              size="lg"
              clip={clips}
              source={sourceMedia}
              preview={playerPreview} />
          </h3>
        </Col>
      </Row>
      <SortableList key="id" list={clips} on:sort={sortList} let:item let:index>
        <Item {item} {index} />
      </SortableList>

    </Col>
    <Col sm="6" class="mx-0 px-0">
      <div class="box">
        <video
          id="video-preview"
          class="video video-js vjs-big-play-centered" />
      </div>
    </Col>
  </Row>
</Container>
