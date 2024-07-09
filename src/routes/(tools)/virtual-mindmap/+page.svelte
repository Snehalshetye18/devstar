<script>
    import { onMount } from 'svelte';
    import Node from '../components/Node.svelte'; // Assuming Node.svelte is in components folder
  
    let nodes = [];
  
    function addNode() {
      nodes = [...nodes, { id: nodes.length, content: 'New Node', top: 100, left: 100 }];
    }
  
    function updateNodePosition(id, top, left) {
      nodes = nodes.map(node => node.id === id ? { ...node, top, left } : node);
    }
  </script>
  
  <div class="mindmap">
    {#each nodes as node (node.id)}
      <Node {node} on:dragMove={e => updateNodePosition(node.id, e.detail.top, e.detail.left)} />
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
  