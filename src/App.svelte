<script>
  let folderRefs = {};
  let currentFolder = null;
  let folders = [...Array(10).keys()].reduce((acc, key) => {
    return [
      ...acc,
      ..."ABCDEFGHIJKLMNOPQRSTUVWXYZ".split("").map(letter => letter + key)
    ];
  }, []);
  let transitionInProgres = false;
  let transtionTimer = null;

  const ANIMATION_DURATION = 350;
  const ANIMATION_DELAY = 10;

  const openStates = new Set(["opening", "opened"]);
  const closeStates = new Set(["closing", null]);

  $: files = currentFolder
    ? [...Array(64).keys()].map(n => `${currentFolder.name} - ${n + 1}`)
    : [];
  $: currentFolderName = currentFolder && currentFolder.name;
  $: folderState = currentFolder ? currentFolder.state : null;

  function beginTransition(beginCallback, endCallback) {
    clearTimeout(transtionTimer);
    transtionTimer = setTimeout(() => {
      transitionInProgres = true;
      beginCallback && beginCallback();
      transtionTimer = setTimeout(() => {
        transitionInProgres = false;
        endCallback && endCallback();
      }, ANIMATION_DURATION);
    }, ANIMATION_DELAY);
  }

  function handleFolderOpen(folderName, event) {
    const parent = {
      width: event.currentTarget.parentElement.offsetWidth,
      height: event.currentTarget.parentElement.offsetHeight,
      scrollLeft: event.currentTarget.parentElement.scrollLeft,
      scrollTop: event.currentTarget.parentElement.scrollTop
    };
    const position = {
      left: event.currentTarget.offsetLeft - parent.scrollLeft,
      top: event.currentTarget.offsetTop - parent.scrollTop,
      width: event.currentTarget.offsetWidth,
      height: event.currentTarget.offsetHeight
    };
    const scaleX = parent.width / position.width;
    const scaleY = parent.height / position.height;
    currentFolder = {
      parent,
      name: folderName,
      scale: Math.min(scaleX, scaleY),
      invertScale: 1 / Math.max(scaleX, scaleY),
      translate: {
        x: parent.width / 2 - (position.left + position.width / 2),
        y: parent.height / 2 - (position.top + position.height / 2)
      },
      state: null
    };
    beginTransition(
      () => {
        currentFolder = {
          ...currentFolder,
          state: "opening"
        };
      },
      () => {
        currentFolder = {
          ...currentFolder,
          state: "opened"
        };
      }
    );
  }

  function handleFolderClose() {
    beginTransition(
      () => {
        currentFolder = {
          ...currentFolder,
          state: "closing"
        };
      },
      () => {
        currentFolder = null;
      }
    );
  }
</script>

<style>
  .container {
    position: relative;
    display: flex;
    width: 100vw;
    height: 100vh;
    background: rgb(30, 31, 34);
    color: silver;
    font-family: Helvetica, Arial, sans-serif;
  }

  .folders,
  .files {
    display: flex;
    justify-content: center;
    align-items: center;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    overflow: auto;
  }

  .folders {
    padding: 10px;
    flex-wrap: wrap;
  }

  .folder {
    position: relative;
    flex: 0 0 auto;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 24px 20px 32px;
    width: 60px;
    height: 40px;
    background: rgb(228, 149, 38);
    cursor: pointer;
    border-radius: 0 4px 4px 4px;
  }

  .folder::before {
    position: absolute;
    content: "";
    background: linear-gradient(
      to bottom,
      rgb(228, 149, 38),
      rgb(173, 106, 12)
    );
    border-radius: 4px 4px 0 0;
    top: -4px;
    left: 0;
    width: 20px;
    height: 4px;
  }

  .folder > p {
    position: absolute;
    bottom: -26px;
    left: 0;
    right: 0;
    font-size: 10px;
    text-align: center;
  }

  .folder,
  .files.transition-enabled {
    transform-origin: 50% 50%;
    transition: opacity 350ms ease, transform 350ms ease;
  }

  .folders:not(.events-enabled),
  .files:not(.events-enabled) {
    pointer-events: none;
  }

  .files {
    flex-direction: column;
    justify-content: flex-start;
    align-items: stretch;
    background-color: rgb(30, 31, 34);
  }

  .file {
    padding: 8px;
    cursor: pointer;
  }

  .file:nth-child(2n) {
    background: rgba(255, 255, 255, 0.05);
  }

  .file:hover {
    background: rgba(255, 255, 255, 0.2);
  }

  .file:nth-child(2n):hover {
    background: rgba(255, 255, 255, 0.25);
  }
</style>

<div class="container">
  <div class="folders" class:events-enabled={folderState === null}>
    {#each folders as folderName}
      <div
        class="folder"
        class:events-enabled={folderState === null || folderState === 'closing'}
        style={currentFolderName === folderName ? (openStates.has(folderState) ? `opacity: 0; transform: translate(${currentFolder.translate.x}px, ${currentFolder.translate.y}px) scale(${currentFolder.scale})` : `opacity: 1; transform: 'none';`) : `opacity: 1; transform: none;`}
        bind:this={folderRefs[folderName]}
        on:click={handleFolderOpen.bind(null, folderName)}>
        <p>{folderName}</p>
      </div>
    {/each}
  </div>
  <div
    class="files"
    on:click={handleFolderClose}
    style={currentFolder && !openStates.has(folderState) ? `opacity: 0; transform: translate(${-currentFolder.translate.x}px, ${-currentFolder.translate.y}px) scale(${currentFolder.invertScale});` : `opacity: ${currentFolder ? 1 : 0}; transform: none;`}
    class:events-enabled={openStates.has(folderState)}
    class:transition-enabled={folderState !== null}>
    {#each files as filename}
      <div class="file">{filename}</div>
    {/each}
  </div>
</div>
