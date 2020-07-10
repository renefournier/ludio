<script>
  import { onMount, onDestroy, createEventDispatcher } from "svelte";
  import SortableList from "svelte-sortable-list";
  import PlayButton from "./components/playButton.svelte";
  import {
    Container,
    Col,
    Row,
    Button
  } from "sveltestrap";
import SourceSelector from "./components/sourceSelector.svelte";
import MarkerButton from "./components/markerButton.svelte";
  import Render from "./components/render.svelte";
  import Clip from "./components/clip.svelte";
  import Item from "./components/item.svelte";
  import Help from "./components/help.svelte";
  import videos from "./data/videos.json";
  import sampleClips from "./data/clips.json";
  // import { createFFmpeg } from "@ffmpeg/ffmpeg";

  // let videos = [],
  //   sampleClips = [];

  // const ffmpeg = createFFmpeg({
  //   log: true
  // });
  window.playerSource = {}; // Attach to window so I can monkey with the player instance from DevToolz
  window.playerPreview = {};

  let playerActive = "source",
    source_id = 0,
    currentTime,
    start = null,
    stop = null,
    elapsed = null,
    showHelp = false,
    clips = [];

let list = [{
  a: 4},{
  b: 68},{
  c: 234},{
  d: 8342
}];

  const  handleKeydown = (event) => {
    // Because the keyboard is mighter [sometimes] than the mouse
    const keyCode = event.keyCode;
    console.log(keyCode);
    if (
      [32, 73, 79, 37, 39, 40, 38, 36, 35, 77, 80, 69, 72].indexOf(keyCode) > -1
    ) {
      event.preventDefault();
    } else {
      return;
    }
    switch (keyCode) {
      case 32: // space: pause/play
        togglePause(); // hold your horses, peoples
        break;
      case 73: // i: trim start
        markIn();
        break;
      case 79: // o: trim stop
        console.log("out");
        markOut();
        break;
      case 37: // left arrow: rwd one frame
        move("←");
        break;
      case 39: // right arrow: fwd one frame
        move("→");
        break;
      case 40: // left arrow: 15 frames
        move("←←");
        break;
      case 38: // right arrow: 15 frames
        move("→→");
        break;
      case 36: // home: go to beginning
        move("⇤");
        break;
      case 35: // end: go to to end
        move("⇥");
        break;
      case 77: // m: toggle mute
        toggleMute();
        break;
      case 80: // e: export
        console.log("Preview clips");
        break;
      case 69: // e: export
        console.log("Export");
        break;
      case 72: // h: toggle help
        showHelp = !showHelp;
        break;
    }
  };

  const togglePause = () => {
    switch (playerActive) {
      case "source":
        if (!playerSource.paused()) {
          playerSource.pause();
        } else {
          playerSource.play();
        }
        break;
      case "preview":
        if (!playerPreview.paused()) {
          playerPreview.pause();
        } else {
          playerPreview.play();
        }
        break;
    }
  };

  const toggleMute = () => {
    // switch (playerActive) {
    //   case "source":
    //     playerSource.muted(!playerSource.muted());
    //     break;
    //   case "preview":
    //     playerPreview.muted(!playerPreview.muted());
    //     break;
    // }

    // Makes more sense for mute to be globa, right?
    playerSource.muted(!playerSource.muted());
    playerPreview.muted(!playerPreview.muted());
  };

  const move = (direction) => {
    // Move playhead
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
        playerSource.pause(); // hold your horses, peoples
        newTime = s - 1 / 30; // one frame
        break;
      case "←←":
        newTime = s - 1 / 2; // 15 frames
        break;
      case "→":
        playerSource.pause(); // hold your horses, peoples
        newTime = s + 1 / 30; // one frame
        break;
      case "→→":
        newTime = s + 1 / 2; // 15 frames
        break;
      case "⇥":
        newTime = duration;
        break;
      case "⇤":
        playerSource.pause(); // hold your horses, peoples
        newTime = 0;
        break;
    }
    console.log(newTime);
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
    const src = playerSource.getMedia().src[0].src;
    if (!start) {
      // do nothing
    } else {
      stop = newTime;
      clips = [
        ...clips,
        { id: Date.now(), src: src, start, stop: stop, duration: stop - start },
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
    clips.forEach((clip) => {
      clipsDurationSeconds += clip.duration;
    });
    clipsDuration = new Date(clipsDurationSeconds * 1000)
      .toISOString()
      .substr(11, 10);
  }

  const play = async (media) => {
    if (typeof media === "object") {
      // Play 'em all — to do
      media.forEach((item) => {
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

  const del = (media) => {
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

  const updateCurrentTime = setInterval(function() {
    const seconds = playerSource.currentTime();
    currentTime = new Date(seconds * 1000).toISOString().substr(11, 10);
    if (start) {
      elapsed = seconds - start;
    }
  }, 100);

  const transcode = async () => {
    console.log("Loading ffmpeg-core.js");
    await ffmpeg.load();
    console.log("Start transcoding");
    const input =
      "https://download-a.akamaihd.net/files/media_publication/a7/pk_E_036_r360P.mp4";
    await ffmpeg.write("temp.mp4", input);
    await ffmpeg.transcode("temp.mp4", "preview.mp4");
    console.log("Complete transcoding");
    const data = ffmpeg.read("preview.mp4");
    playerPreview.src({
      src: URL.createObjectURL(new Blob([data.buffer], { type: "video/mp4" })),
    });
  };

  const sortList = (ev) => {
    console.log (ev.detail)
    // list = ev.detail;
  };

  const changeSource = async (src_id) => {
    console.log(src_id);
    await playerSource.pause();
    await playerSource.src(videos[src_id]);
    await playerSource.play();
  };

  onMount(async () => {
    console.log("M: 🏇");

    clips = sampleClips.map((clip) => {
      // add random sample sourcemedia
      clip.src = videos[Math.floor(Math.random() * videos.length)];
      return clip;
    });

    window.playerSource = videojs(
      "video-source",
      {
        autoplay: false,
        controls: true,
        sources: videos[source_id], // let’s pick one at random, shall we? It’s all great content, so let’s not be too picky, right?!
      },
      function onPlayerReady() {
        console.log("onPlayerReady0");
        // playerSource.muted(true);
      }
    );

    window.playerPreview = videojs(
      "video-preview",
      {
        autoplay: false,
        controls: true,
        sources: videos[Math.floor(Math.random() * videos.length)],
      },
      function onPlayerReady() {
        console.log("onPlayerReady1");
        // playerPreview.muted(true);
      }
    );

    playerSource.on("play", function(e) {
      playerActive = "source";
      playerPreview.pause();
    });

    playerPreview.on("play", function(e) {
      playerActive = "preview";
      playerSource.pause();
    });

    // transcode();
  });

  onDestroy(async () => {
    console.log("U: 🐎 "); // rider falls off
  });
</script>

<svelte:window on:keydown={handleKeydown} />

<Container fluid class="px-0">
  <Row class="mx-0 px-0 w-100">
    <Col sm="6" class="mx-0 px-0">

      <Row class="m-0 p-0 monitor">
        <Col />
        <Col>
          <p class:active={playerActive === 'source'} class="px-4 py-2 mb-0 ">
            <span class="fa fa-sign-in-alt mr-1" />
            <strong>Source</strong>
          </p>
        </Col>
        <Col>
          <SourceSelector {videos} {source_id} {changeSource} />          
        </Col>
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

        <SortableList key="id" list={clips} let:item let:index>
          <Item
            {item}
            {index}
            on:sort={sortList}
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

        <Col sm="6" class="mx-0 px-2 text-right">
          <div class="btn " on:click={() => (showHelp = !showHelp)}>
            <i class="fa fa-lg fa-question-circle " />
          </div>
        </Col>
      </Row>

      {#if showHelp}
        <Row class="mx-0 px-0 w-100">
          <Col sm="12" class="mx-0 p-5">
            <Help />
          </Col>
        </Row>
      {/if}

    </Col>
  </Row>

  <Row class="mx-0 px-0 w-100" style="border-left: 1px solid rgba(0,0,0,1)">
    <Col class="p-2">
      <code>{JSON.stringify(clips)}</code>
    </Col>

  </Row>
</Container>

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
    color: #ffffff;
    text-shadow: 0 0 10px rgba(0, 0, 0, 0.25);
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

  .btn > * {
    color: rgba(255, 255, 255, 0.25);
    cursor: pointer;
  }

  .btn:hover > * {
    color: #00b4ff !important;
  }
</style>
