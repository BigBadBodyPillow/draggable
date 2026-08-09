<script lang="ts">
  interface Coordinates {
    x: number;
    y: number;
  }

  let tag: HTMLElement | null = $state(null);
  let section: HTMLElement | null = $state(null);
  let offset: Coordinates = $state({ x: 0, y: 0 });
  let isDragging = $state(false);
  let dragOffset: Coordinates = $state({ x: 0, y: 0 });
  let sectionCenter: Coordinates = $state({ x: 0, y: 0 });
  let maxDrag: Coordinates = $state({ x: 0, y: 0 });
  let timing: string = $state('cubic-bezier(0,1,1,1)');

  const normalTiming = 'cubic-bezier(0,1,0,1)';
  const bounceOutThenInTiming = 'cubic-bezier(.38,2.48,.74,-0.38)';

  function clamp(value: number, min: number, max: number) {
    return Math.min(Math.max(value, min), max);
  }

  function updateTiming() {
    const distance = Math.hypot(offset.x, offset.y);
    timing = distance > 100 ? bounceOutThenInTiming : normalTiming;
  }

  function getCursorPosition(e: MouseEvent) {
    return { x: e.clientX, y: e.clientY };
  }

  function updateDragLimits() {
    if (!section || !tag) return;

    const sectionRect = section.getBoundingClientRect();
    const tagRect = tag.getBoundingClientRect();
    const padding = 20; /* limit */

    sectionCenter = {
      x: Math.round(sectionRect.left + sectionRect.width / 2),
      y: Math.round(sectionRect.top + sectionRect.height / 2)
    };

    maxDrag = {
      x: Math.round((sectionRect.width - tagRect.width) / 2 - padding),
      y: Math.round((sectionRect.height - tagRect.height) / 2 - padding)
    };
  }

  function startDragging(e: MouseEvent) {
    if (!tag) return;

    e.preventDefault();
    updateDragLimits();
    const cursor = getCursorPosition(e);

    const rect = tag.getBoundingClientRect();
    const centerX = rect.left + rect.width / 2;
    const centerY = rect.top + rect.height / 2;

    dragOffset = {
      x: cursor.x - centerX,
      y: cursor.y - centerY
    };

    timing = normalTiming;
    isDragging = true;
  }

  function stopDragging() {
    if (!isDragging) return;
    isDragging = false;
    resetOffset();
  }

  function moveElement(e: MouseEvent) {
    if (!isDragging) return;

    const cursor = getCursorPosition(e);
    const desiredCenterX = cursor.x - dragOffset.x;
    const desiredCenterY = cursor.y - dragOffset.y;

    offset = {
      x: Math.round(clamp(desiredCenterX - sectionCenter.x, -maxDrag.x, maxDrag.x)),
      y: Math.round(clamp(desiredCenterY - sectionCenter.y, -maxDrag.y, maxDrag.y))
    };

    updateTiming();
  }

  function resetOffset() {
    offset = {
      x: 0,
      y: 0
    };
  }
</script>

<svelte:window
  onmousemove={isDragging ? moveElement : undefined}
  onmouseup={stopDragging}
  onmouseleave={stopDragging}
/>

<section role="main" bind:this={section}>
  <button
    id="tag"
    bind:this={tag}
    style={`--x:${offset.x}px; --y:${offset.y}px; ${isDragging ? 'cursor: grabbing;' : `transition: all 0.3s ${timing}`}`}
    // cubic-bezier(.38,2.48,.74,-0.38)
    // cubic-bezier(0,2.64,.31,.35)
    // cubic-bezier(0,1.5,1,1)
    onmousedown={startDragging}
    aria-label="draggable"
  >
  </button>
</section>

<style>
  section {
    max-width: 700px;
    min-width: 200px;
    width: 100%;
    aspect-ratio: 1.6;
    background-color: antiquewhite;

    display: flex;
    justify-content: center;
    align-items: center;
  }

  #tag {
    --x: 0;
    --y: 0;

    width: 60px;
    aspect-ratio: 1;
    border-radius: 100vh !important;
    border: none;
    background-color: var(--bg);
    color: var(--text);
    padding: 0.6rem 0.8rem;
    cursor: grab;
    user-select: none;

    transform: translate(var(--x), var(--y));
    transition:
      background-color 0.2s,
      color 0.2s;
    will-change: transform;
  }
</style>
