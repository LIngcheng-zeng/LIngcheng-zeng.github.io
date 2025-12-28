

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <title>@antv/g6 Tutorial</title>
    <style>
      html, body, #container {
        height: 100%;
        margin: 0;
        padding: 0;
      }
      #search-wrap {
        position: absolute;
        top: 12px;
        right: 12px;
        z-index: 9999;
        display: flex;
        align-items: center;
        gap: 8px;
        background: rgba(255,255,255,0.9);
        padding: 6px 8px;
        border-radius: 6px;
        box-shadow: 0 2px 8px rgba(0,0,0,0.12);
        font-family: sans-serif;
      }
      #search-input {
        min-width: 220px;
        padding: 6px 8px;
        border: 1px solid #ddd;
        border-radius: 4px;
      }
      #search-count {
        font-size: 12px;
        color: #666;
      }
    </style>
  </head>
  <body>
    <div id="container"></div>

    <div id="search-wrap">
      <input id="search-input" placeholder="Search nodes (Ctrl+F)" />
      <div id="search-count"></div>
    </div>

    <script type="module" src="main.ts"></script>
  </body>
</html>
```



```ts
import { Graph, treeToGraphData } from '@antv/g6';

/**
 * If the node is a leaf node
 * @param {*} d - node data
 * @returns {boolean} - whether the node is a leaf node
 */
function isLeafNode(d) {
  return !d.children || d.children.length === 0;
}

fetch('https://gw.alipayobjects.com/os/antvdemo/assets/data/algorithm-category.json')
  .then((res) => res.json())
  .then((data) => {
    const graph = new Graph({
      container: 'container',
      autoFit: 'view',
      data: treeToGraphData(data),
      node: {
        style: {
          labelText: (d) => d.id,
          labelPlacement: (d) => (isLeafNode(d) ? 'right' : 'left'),
          labelBackground: true,
          ports: [{ placement: 'right' }, { placement: 'left' }],
        },
        animation: {
          enter: false,
        },
        // G6 v5 uses `state` to style node states
        state: {
          highlight: {
            stroke: '#f5222d',
            lineWidth: 3,
            shadowColor: 'rgba(245,34,45,0.18)',
            shadowBlur: 12,
          },
          inactive: {
            opacity: 0.28,
          },
        },
      },
      edge: {
        type: 'cubic-horizontal',
        animation: {
          enter: false,
        },
      },
      layout: {
        type: 'dendrogram',
        direction: 'LR', // H / V / LR / RL / TB / BT
        nodeSep: 36,
        rankSep: 250,
      },
      behaviors: ['drag-canvas', 'zoom-canvas', 'drag-element', 'collapse-expand'],
    });

    graph.render();

    // --- Search & highlight (G6 v5 compatible) ---
    const input = document.getElementById('search-input') as HTMLInputElement | null;
    const countEl = document.getElementById('search-count');

    function clearStates() {
      // only remove highlight state from previously highlighted nodes
      try {
        const prev = graph.getElementDataByState('node', 'highlight') || [];
        prev.forEach((n: any) => {
          graph.setElementState(n.id, [] as any).catch(() => {});
        });
      } catch (e) {
        // ignore
      }
      if (countEl) countEl.textContent = '';
    }

    function searchNodes(query: string) {
      const q = (query || '').trim().toLowerCase();
      const nodesData = graph.getNodeData();
      if (!q) {
        clearStates();
        return;
      }
      const matchIds: string[] = [];
      let firstMatchId: string | null = null;
      let matchCount = 0;
      nodesData.forEach((node: any) => {
        const text = (node.id || '').toString();
        const matched = text.toLowerCase().includes(q);
        if (matched) {
          matchIds.push(node.id);
          matchCount++;
          if (!firstMatchId) firstMatchId = node.id;
        }
      });

      // remove highlight from previously highlighted nodes that are not matched
      try {
        const prev = graph.getElementDataByState('node', 'highlight') || [];
        prev.forEach((n: any) => {
          if (!matchIds.includes(n.id)) {
            graph.setElementState(n.id, [] as any).catch(() => {});
          }
        });
      } catch (e) {
        // ignore
      }

      // add highlight to matched nodes (leave others untouched)
      matchIds.forEach((id) => {
        graph.setElementState(id, 'highlight' as any).catch(() => {});
      });

      if (countEl) countEl.textContent = `${matchCount}`;
      if (firstMatchId) {
        graph.focusElement(firstMatchId).catch(() => {});
      }
    }

    if (input) {
      input.addEventListener('input', (ev) => {
        const v = (ev.target as HTMLInputElement).value;
        searchNodes(v);
      });
      input.addEventListener('keydown', (ev) => {
        if (ev.key === 'Escape') {
          input.value = '';
          clearStates();
          input.blur();
        }
      });
    }

    // Intercept Ctrl/Cmd+F to focus our search input
    window.addEventListener('keydown', (ev) => {
      const isFind = (ev.ctrlKey || ev.metaKey) && ev.key.toLowerCase() === 'f';
      if (isFind) {
        ev.preventDefault();
        if (input) {
          input.focus();
          input.select();
        }
      }
      if (ev.key === 'Escape') {
        if (input && document.activeElement === input) {
          input.value = '';
          clearStates();
        }
      }
    });
  });
```



```ts
{
  "name": "g6",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
     "dev": "vite",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs",
  "dependencies": {
    "@antv/g6": "^5.0.50"
  },
  "devDependencies": {
    "vite": "^7.3.0"
  }
}

```

