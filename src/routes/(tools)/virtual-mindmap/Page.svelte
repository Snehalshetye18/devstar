<script>
  import { onMount } from 'svelte';

  let nodes = [];
  let connections = [];
  let selectedNodes = [];

  function addNode() {
    nodes = [...nodes, { id: nodes.length, content: 'New Node', top: 100, left: 100 }];
  }

  function updateNodePosition(id, top, left) {
    nodes = nodes.map(node => node.id === id ? { ...node, top, left } : node);
    updateConnections();
  }

  function allowDrop(event) {
    event.preventDefault();
  }

  function handleDrop(event) {
    const data = JSON.parse(event.dataTransfer.getData('application/json'));
    const { id, offsetX, offsetY } = data;
    const top = event.clientY - offsetY;
    const left = event.clientX - offsetX;
    updateNodePosition(id, top, left);
  }

  function handleDragStart(event, node) {
    event.dataTransfer.setData('application/json', JSON.stringify({ id: node.id, offsetX: event.offsetX, offsetY: event.offsetY }));
  }

  function connectNodes() {
    if (selectedNodes.length === 2) {
      const [id1, id2] = selectedNodes;
      connections = [...connections, { id1, id2 }];
      updateConnections();
      selectedNodes = []; // Reset selection after connecting
    }
  }

  function updateConnections() {
    connections = connections.map(conn => {
      const node1 = nodes.find(node => node.id === conn.id1);
      const node2 = nodes.find(node => node.id === conn.id2);
      if (node1 && node2) {
        return {
          ...conn,
          x1: node1.left + 50, // 50 is half the node width to center the line
          y1: node1.top + 25, // 25 is half the node height to center the line
          x2: node2.left + 50,
          y2: node2.top + 25,
        };
      }
      return conn;
    });
  }

  function toggleSelectNode(id) {
    if (selectedNodes.includes(id)) {
      selectedNodes = selectedNodes.filter(nodeId => nodeId !== id);
    } else {
      if (selectedNodes.length < 2) {
        selectedNodes = [...selectedNodes, id];
      }
    }
  }

  // Example to connect the first two nodes after mounting
  onMount(() => {
    if (nodes.length > 1) {
      connectNodes(0, 1);
      updateConnections();
    }
  });
</script>

<div class="mindmap" on:dragover={allowDrop} on:drop={handleDrop}>
  {#each nodes as node (node.id)}
    <div 
      class="node {selectedNodes.includes(node.id) ? 'selected' : ''}" 
      style="top: {node.top}px; left: {node.left}px;" 
      draggable="true" 
      on:dragstart={(event) => handleDragStart(event, node)}
      on:click={() => toggleSelectNode(node.id)}>
      {node.content}
    </div>
  {/each}

  <svg class="connections" width="100%" height="100%">
    {#each connections as { x1, y1, x2, y2 }}
      <line x1={x1} y1={y1} x2={x2} y2={y2} stroke="black" />
    {/each}
  </svg>
</div>
<button on:click={addNode}>Add Node</button>
<button on:click={connectNodes} disabled={selectedNodes.length !== 2}>Connect Nodes</button>

<style>
  .mindmap {
    position: relative;
    width: 100%;
    height: 100vh;
    background-color: #f0f0f0;
  }
  .node {
    position: absolute;
    width: 100px;
    height: 50px;
    padding: 10px;
    background-color: #fff;
    border: 1px solid #ccc;
    border-radius: 5px;
    cursor: move;
    text-align: center;
    line-height: 50px;
  }
  .node.selected {
    border-color: blue;
    box-shadow: 0 0 10px blue;
  }
  .connections {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
  }
  button {
    margin: 10px;
  }
</style>
