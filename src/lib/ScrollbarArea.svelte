<!--suppress CheckEmptyScriptTag -->
<script lang="ts">
  import { onDestroy, onMount } from 'svelte';
  import { nanoid } from 'nanoid';

  let contentArea: HTMLDivElement;
  let scrollbarTrack: HTMLDivElement;
  let scrollbarThumb: HTMLDivElement;
  let observer: ResizeObserver;

  let thumbHeight = 32;
  let isDragging = false;
  let scrollStartPosition = 0;
  let initialScrollTop = 0;
  let scrollPosition = 0;
  const viewportID = nanoid();

  function handleWheel(evt: WheelEvent) {
    evt.preventDefault();
    evt.stopPropagation();
    if (contentArea) {
      contentArea.scrollBy({
        top: evt.deltaY,
        behavior: 'smooth'
      });
    }
  }

  function handleThumbResize(ref: HTMLDivElement, trackSize: number) {
    const { clientHeight, scrollHeight } = ref;
    thumbHeight = Math.max((clientHeight / scrollHeight) * trackSize, 32);
    thumbHeight = thumbHeight;
  }

  function handleThumbPosition() {
    const { scrollTop: contentTop, scrollHeight: contentHeight } = contentArea;
    const { clientHeight: trackHeight } = scrollbarTrack;
    const newTop = Math.min((contentTop / contentHeight) * trackHeight, trackHeight - thumbHeight);
    scrollbarThumb.style.top = `${newTop}px`;
    scrollPosition = (scrollbarThumb.offsetTop / Math.floor(trackHeight - thumbHeight)) * 100;
  }

  function handleTrackClick(evt: MouseEvent) {
    evt.preventDefault();
    evt.stopPropagation();

    const { clientY } = evt;
    const target = evt.target as HTMLDivElement;
    const rect = target.getBoundingClientRect();
    const trackTop = rect.top;
    const thumbOffset = -(thumbHeight / 2);
    const clickRatio = (clientY - trackTop + thumbOffset) / scrollbarTrack.clientHeight;
    const scrollAmount = Math.floor(clickRatio * contentArea.scrollHeight);
    contentArea.scrollTo({ top: scrollAmount, behavior: 'smooth' });
  }

  function handleThumbPointerDown(evt: PointerEvent) {
    if (evt.pointerType === 'mouse' && evt.button !== 0) {
      return;
    }
    evt.preventDefault();
    evt.stopPropagation();
    scrollStartPosition = evt.clientY;
    initialScrollTop = contentArea.scrollTop;
    isDragging = true;
    isDragging = isDragging;
  }

  function handleThumbPointerUpOrLeave(evt: PointerEvent) {
    if (!(evt.pointerType === 'mouse' && evt.button !== 0) || evt.type === 'pointerleave') {
      evt.preventDefault();
      evt.stopPropagation();
      if (isDragging) {
        isDragging = false;
        isDragging = isDragging;
      }
    }
  }

  function handleThumbPointerMove(evt: PointerEvent) {
    if (isDragging) {
      evt.preventDefault();
      evt.stopPropagation();

      const { scrollHeight: contentScrollHeight, offsetHeight: contentOffsetHeight } = contentArea;
      const deltaY = (evt.clientY - scrollStartPosition) * (contentOffsetHeight / thumbHeight);
      contentArea.scrollTop = Math.min(
        initialScrollTop + deltaY,
        contentScrollHeight - contentOffsetHeight
      );
    }
  }

  onMount(() => {
    const { clientHeight: trackSize } = scrollbarTrack;
    observer = new ResizeObserver(() => {
      handleThumbResize(contentArea, trackSize);
    });
    observer.observe(contentArea);
  });

  onDestroy(() => {
    observer.unobserve(contentArea);
  });
</script>

<div
  class="ScrollbarArea"
  on:wheel={handleWheel}
  on:pointerup={handleThumbPointerUpOrLeave}
  on:pointerleave={handleThumbPointerUpOrLeave}
  on:pointermove={handleThumbPointerMove}
>
  <div class="ScrollbarArea__wrapper">
    <div
      class="ScrollbarArea__viewport"
      id={viewportID}
      bind:this={contentArea}
      on:scroll={handleThumbPosition}
      role="document"
    >
      <slot />
    </div>
  </div>
  <div
    class="ScrollbarArea__scrollbar"
    role="scrollbar"
    aria-controls={viewportID}
    aria-valuenow={scrollPosition}
  >
    <div
      class="ScrollbarArea__scrollbar-track"
      bind:this={scrollbarTrack}
      on:click={handleTrackClick}
    />
    <div
      class="ScrollbarArea__scrollbar-thumb"
      style="height: {thumbHeight}px; cursor: {isDragging ? 'grabbing' : 'grab'}"
      bind:this={scrollbarThumb}
      on:pointerdown={handleThumbPointerDown}
    />
  </div>
</div>

<style lang="scss">
  .ScrollbarArea {
    position: relative;
    display: grid;
    grid-template: auto / 1fr auto;
    gap: 0.5rem;
    width: 60%;
    height: 60vh;
    border-radius: 10px;
    margin: 0 auto;

    &__wrapper {
      max-height: 100%;
      padding: 0.25rem;
      overflow: hidden;
      background-color: #fafafe;
      border: 1px solid gray;
      border-radius: 10px;
    }

    &__viewport {
      position: relative;
      height: 100%;
      display: flex;
      flex-direction: column;
      border-radius: 10px;
      overflow: auto;
      -ms-overflow-style: none;

      // Only supported by Mozilla browsers at the moment
      scrollbar-width: none;

      &::-webkit-scrollbar {
        display: none;
      }
    }

    &__scrollbar {
      position: relative;
      border-radius: 10px;
      display: grid;
      height: 100%;
      width: 1rem;

      &-track {
        position: absolute;
        bottom: 0;
        top: 0;
        width: 100%;
        background-color: lightgrey;
        border-radius: 10px;
        cursor: pointer;
        outline: 1px solid darkgrey;
      }

      &-thumb {
        position: absolute;
        width: 100%;
        border-radius: 10px;
        background-color: darkcyan;
        outline: 1px solid darkgreen;
        height: 2rem;
      }
    }
  }
</style>
