<script>
    import { onMount } from 'svelte';
    import Node from '../components/Node.svelte';
  
    let nodes = [];
  
    function addNode() {
      nodes = [...nodes, { id: nodes.length, content: 'New Node', top: 100, left: 100 }];
    }
  
    function updateNodePosition(id, top, left) {
      nodes = nodes.map(node => node.id === id ? { ...node, top, left } : node);
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
  </script>
  
  <div class="mindmap" on:dragover={allowDrop} on:drop={handleDrop}>
    {#each nodes as node (node.id)}
      <Node {node} />
    {/each}
  </div>
  <button on:click={addNode}>Add Node</button>
  
  <style>
    .mindmap {
      position: relative;
      width: 100%;
      height: 100vh;
      background-color: #f0f0f0;
    }
  </style>
  