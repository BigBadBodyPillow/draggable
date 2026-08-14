<script lang="ts">
  import arrow from '../assets/arrow.svg?raw';

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
  let rotation: number = $state(0);

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

  function handleMouseMove(e: MouseEvent) {
    const sectionRect = section?.getBoundingClientRect();
    if (!sectionRect) return;
    const radius = sectionRect?.height / 2; /* radiusof rectangle? idk what its called */

    const topCenter = { x: sectionCenter.x, y: sectionCenter.y - radius };
    const cursor = getCursorPosition(e);

    rotation = getAngle(sectionCenter, cursor);

    if (isDragging) {
      moveElement(e);
    }
  }

  // function getAngle(
  //   PointWhereTheAngleIs: Coordinates,
  //   opositePoint1: Coordinates,
  //   opositePoint2: Coordinates
  // ) {
  //   // a^2 = b^2 + c^2 -2bc * cos(A)
  //   // 2bc * cos(A) =b^2 + c^2 - a^2
  //   // A = arccos((b^2 + c^2 -a^2)/2bc)
  //   const a = getDistance(opositePoint1, opositePoint2);
  //   const b = getDistance(PointWhereTheAngleIs, opositePoint1);
  //   const c = getDistance(PointWhereTheAngleIs, opositePoint2);

  //   const radians = Math.acos((b ** 2 + c ** 2 - a ** 2) / (2 * b * c));
  //   const angle = radians * (180 / Math.PI);

  //   return angle;
  // }

  function getAngle(center: Coordinates, target: Coordinates) {
    const dx = target.x - center.x;
    const dy = target.y - center.y;

    // negative y because css higher number = lower down
    const angle = Math.atan2(dx, -dy) * (180 / Math.PI);

    return angle;
  }

  // function getDistance(pointOne: Coordinates, PointTwo: Coordinates) {
  //   // distance = root(x2 - x1)^2 + (y2-y1)^2
  //   const x = PointTwo.x - pointOne.x;
  //   const y = PointTwo.y - pointOne.y;
  //   const distance = Math.sqrt(x ** 2 + y ** 2);
  //   return distance;
  // }

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
  onmousemove={handleMouseMove}
  onmouseup={stopDragging}
  onmouseleave={stopDragging}
  onresize={updateDragLimits}
/>

<section role="main" bind:this={section} {@attach updateDragLimits}>
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
    <div class="arrow" style={`--rotation:${rotation}deg`}>{@html arrow}</div>
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
    --angle: 0deg;
    --radius: 30px;

    position: relative;

    width: calc(var(--radius) * 2);
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

  .arrow {
    --width: 20px;
    --offset: 13px;
    --rotation: 0deg;
    position: absolute;
    top: 50%;
    left: 50%;
    /* diameter of circle + arrows height + the offset(*2 becuase  its centered in center so the variable would be halved) */
    height: calc(var(--radius) * 2 + (var(--offset) * 2) + var(--width));
    transform-origin: center center;
    transform: translate(-50%, -50%) rotate(var(--rotation));

    display: flex;
    justify-content: center;
    align-items: start;
    box-sizing: border-box;
    border-radius: 0px;

    :global(svg) {
      width: var(--width);
      height: var(--width);
      fill: red;
    }
  }
</style>
