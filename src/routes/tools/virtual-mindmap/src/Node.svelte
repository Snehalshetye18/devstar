<script>
    export let node;
    let top = node.top;
    let left = node.left;
  
    function handleDragStart(event) {
      event.dataTransfer.setData('application/json', JSON.stringify({ id: node.id, offsetX: event.offsetX, offsetY: event.offsetY }));
    }
  
    function handleDrop(event) {
      const data = JSON.parse(event.dataTransfer.getData('application/json'));
      top = event.clientY - data.offsetY;
      left = event.clientX - data.offsetX;
      dispatch('dragMove', { id: node.id, top, left });
    }
  
    function dispatch(event, detail) {
      const customEvent = new CustomEvent(event, { detail });
      dispatchEvent(customEvent);
    }
  </script>
  
  <div class="node" style="top: {top}px; left: {left}px;" draggable="true" on:dragstart={handleDragStart} on:drop={handleDrop}>
    {node.content}
  </div>
  
  <style>
    .node {
      position: absolute;
      padding: 10px;
      background-color: #fff;
      border: 1px solid #ccc;
      border-radius: 5px;
      cursor: move;
    }
  </style>
  