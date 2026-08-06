<script lang="ts">
  import { onMount } from 'svelte';

  let tag: HTMLDivElement | null = $state(null);

  function getElementPosition(element: HTMLElement | null) {
    if (!element) {
      return { x: 0, y: 0 };
    }

    const rect = element.getBoundingClientRect();
    return {
      x: rect.left,
      y: rect.top
    };
  }

  function moveElement(element: HTMLElement | null, newX: number, newY: number) {
    if (element) {
      // const { x, y } = getElementPosition(element);

      element.style.transform = `translate(${newX}px, ${newY}px)`;
    }
  }

  onMount(() => {
    // const { x, y } = getElementPosition(tag);
    // console.log('Initial position:', { x, y });
    moveElement(tag, 30, 0);
  });
</script>

<section id="window">
  <div id="tag" bind:this={tag} data-X="" data-Y="">Drag Me</div>
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
    background-color: var(--bg);
    padding: 0.6rem 0.8rem;
    border-radius: var(--radius);
    cursor: pointer;
    user-select: none;
  }
</style>
