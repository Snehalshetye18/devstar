<script>
  import { writable } from "svelte/store";
  import html2canvas from "html2canvas";

  export let nodes = writable([
    {
      id: 1,
      text: "Main Topic",
      x: 500,
      y: 300,
      parentId: null,
      color: "#ffffff",
      locked: false,
      lineColor: "#007BFF", // Set initial line color to blue
    },
  ]);

  let nextId = 2;
  let dragNode = null;
  let newParent = null;
  let title = "MindMap";
  const lineThickness = 4;
  const directions = [
    [-150, 0],
    [150, 0],
    [0, -150],
    [0, 150],
    [-150, 150],
    [150, -150],
  ];
  let directionIndex = 0;

  function addNode(parentId) {
    nodes.update((n) => {
      const parentNode = n.find((node) => node.id === parentId);
      const newX = parentNode.x + directions[directionIndex][0] + 75;
      const newY = parentNode.y + directions[directionIndex][1] + 75;
      directionIndex = (directionIndex + 1) % directions.length;
      return [
        ...n,
        {
          id: nextId++,
          text: `Node ${nextId}`,
          x: newX,
          y: newY,
          parentId: parentId,
          color: "#ffffff",
          locked: false,
          lineColor: "#007BFF", // Set initial line color to blue
        },
      ];
    });
  }

  function updateText(node, newText) {
    nodes.update((n) => {
      const targetNode = n.find((n) => n.id === node.id);
      if (targetNode) {
        targetNode.text = newText;
      }
      return [...n];
    });
  }

  function deleteNode(nodeId) {
    nodes.update((n) => {
      const remainingNodes = n.filter(
        (node) => node.id !== nodeId && node.parentId !== nodeId
      );
      return remainingNodes.length > 0 ? remainingNodes : n;
    });
  }

  function handleMouseDown(event, node) {
    if (!node.locked) {
      dragNode = node;
    }
  }

  function handleMouseMove(event) {
    if (dragNode) {
      const container = document.querySelector(".mindmap-container");
      const containerRect = container.getBoundingClientRect();

      let newX = event.clientX - containerRect.left;
      let newY = event.clientY - containerRect.top;

      newX = Math.max(0, Math.min(newX, containerRect.width - 150));
      newY = Math.max(0, Math.min(newY, containerRect.height - 50));

      dragNode.x = newX;
      dragNode.y = newY;
      nodes.update((n) => [...n]);
    }
  }

  function handleMouseUp() {
    dragNode = null;
  }

  function setNewParent(node) {
    newParent = node;
  }

  function changeParent(nodeId) {
    if (newParent) {
      nodes.update((n) => {
        const targetNode = n.find((node) => node.id === nodeId);
        if (targetNode) {
          targetNode.parentId = newParent.id;
        }
        return [...n];
      });
      newParent = null;
    }
  }

  function updateLineColor(node, newColor) {
    nodes.update((n) => {
      const targetNode = n.find((n) => n.id === node.id);
      if (targetNode) {
        targetNode.lineColor = newColor;
      }
      return [...n];
    });
  }

  async function downloadMindmap() {
    const mindmapContainer = document.querySelector(".mindmap-container");

    const canvas = await html2canvas(mindmapContainer);
    const pngUrl = canvas.toDataURL("image/png");

    const downloadLink = document.createElement("a");
    downloadLink.href = pngUrl;
    downloadLink.download = title;
    document.body.appendChild(downloadLink);
    downloadLink.click();
    document.body.removeChild(downloadLink);
  }

  function toggleLock(node) {
    nodes.update((n) => {
      const targetNode = n.find((n) => n.id === node.id);
      if (targetNode) {
        targetNode.locked = !targetNode.locked;
      }
      return [...n];
    });
  }

  let zoomLevel = 1;

  function zoomIn() {
    zoomLevel = Math.min(zoomLevel + 0.1, 2);
  }

  function zoomOut() {
    zoomLevel = Math.max(zoomLevel - 0.1, 0.5);
  }
</script>

<div class="header">
  <div class="title">
    <h1 contenteditable bind:textContent={title}>MindMap</h1>
  </div>
  <div class="actions">
    <button on:click={downloadMindmap}>Download</button>
  </div>
  <div class="zoom-controls">
    <button on:click={zoomIn}>Zoom In</button>
    <button on:click={zoomOut}>Zoom Out</button>
  </div>
</div>

<div
  class="mindmap-container"
  on:mousemove={handleMouseMove}
  on:mouseup={handleMouseUp}
  style="--zoom-level: {zoomLevel};"
>
  <div class="scaling-content">
    <svg class="absolute w-full h-full">
      {#each $nodes as node (node.id)}
        {#if node.parentId !== null}
          {#each $nodes as parent (parent.id)}
            {#if parent.id === node.parentId}
              <line
                x1={parent.x + 75}
                y1={parent.y + 25}
                x2={node.x + 75}
                y2={node.y + 25}
                style="stroke: {node.lineColor}; stroke-width: {lineThickness}px;"
              ></line>
            {/if}
          {/each}
        {/if}
      {/each}
    </svg>
    {#each $nodes as node (node.id)}
      <div
        class="node"
        style="left: {node.x}px; top: {node.y}px; background-color: {node.color};"
        on:mousedown={(e) => handleMouseDown(e, node)}
      >
        <textarea
          value={node.text}
          on:input={(e) => updateText(node, e.target.value)}
          readonly={node.locked}
        />
        <div class="panel">
          <svg
            on:click={() => addNode(node.id)}
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="size-6 add-icon"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M12 4.5v15m7.5-7.5h-15"
            />
          </svg>
          <svg
            on:click={() => deleteNode(node.id)}
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="currentColor"
            class="size-6 delete-icon"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0"
            />
          </svg>
          {#if node.locked}
            <svg
              on:click={() => toggleLock(node)}
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="size-6 unlock-icon"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M13.5 10.5V6.75a4.5 4.5 0 1 1 9 0v3.75M3.75 21.75h10.5a2.25 2.25 0 0 0 2.25-2.25v-6.75a2.25 2.25 0 0 0-2.25-2.25H3.75a2.25 2.25 0 0 0-2.25 2.25v6.75a2.25 2.25 0 0 0 2.25 2.25Z"
              />
            </svg>
          {:else}
            <svg
              on:click={() => toggleLock(node)}
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
              stroke-width="1.5"
              stroke="currentColor"
              class="size-6 lock-icon"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M16.5 10.5V6.75a4.5 4.5 0 0 0-9 0v3.75m-.75 11.25h10.5a2.25 2.25 0 0 0 2.25-2.25v-6.75a2.25 2.25 0 0 0-2.25-2.25H6.75a2.25 2.25 0 0 0-2.25 2.25v6.75a2.25 2.25 0 0 0 2.25 2.25Z"
              />
            </svg>
          {/if}
          <input
            type="color"
            bind:value={node.lineColor}
            on:input={(e) => updateLineColor(node, e.target.value)}
          />
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  .panel {
    display: none;
    position: absolute;
    top: 40px;
    right: 5px;
    background-color: #fff;
    border: 1px solid #ccc;
    padding: 5px;
    z-index: 1000;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
    opacity: 0.8;
    transition: opacity 0.3s ease;
    font-size: 12px;
  }

  .node:hover .panel {
    display: block;
    opacity: 1;
  }

  .node {
    position: absolute;
    width: 150px;
    height: 50px;
    background-color: #ffffff;
    border: 1px solid #000000;
    border-radius: 5px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .node:hover {
    background-color: #f0f0f0;
  }

  .node textarea {
    width: 100%;
    height: 100%;
    border: none;
    resize: none;
    outline: none;
    overflow: hidden;
    background: transparent;
    text-align: center;
    font-size: 14px;
    font-family: Arial, Helvetica, sans-serif;
  }

  .node .panel svg {
    margin: 2px;
    cursor: pointer;
    width: 18px;
    height: 18px;
  }

  .absolute {
    position: absolute;
    top: 0;
    left: 0;
  }

  line {
    pointer-events: none;
    z-index: -1; /* Ensuring lines are behind the nodes */
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px;
    background-color: #f0f0f0;
    border-bottom: 1px solid #ccc;
  }

  .title h1 {
    margin: 0;
    font-size: 24px;
  }

  .actions button {
    padding: 8px 16px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  .actions button:hover {
    background-color: #0056b3;
  }

  .mindmap-container {
    position: relative;
    width: 100%;
    height: calc(100vh - 80px);
    overflow: auto;
    background-color: #f5f5f5;
    cursor: grab;
  }

  .scaling-content {
    transform: scale(var(--zoom-level));
    transform-origin: top left;
    transition: transform 0.3s ease;
  }

  .zoom-controls button {
    padding: 8px 16px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-right: 5px;
  }

  .zoom-controls button:hover {
    background-color: #0056b3;
  }
</style>
