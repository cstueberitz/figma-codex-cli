# Codex Working Notes

## Project Overview

`figma-ds-cli` is a local CLI for working with Figma design files. It connects through Chrome DevTools Protocol and runs JavaScript against the Figma Plugin API.

**Location:** Repo root (the directory containing `src/index.js`)
**npm package:** `figma-ds-cli` (v1.1.1)
**GitHub:** https://github.com/cstueberitz/figma-codex-cli

## Useful Commands

### Execute JavaScript in Figma

```bash
node src/index.js eval "YOUR_JAVASCRIPT_HERE"
```

### Query Nodes

```bash
node src/index.js raw query "//FRAME"
node src/index.js raw query "//GROUP[@name='content']"
node src/index.js raw query "//*[@name^='session-']"
```

### Export

```bash
node src/index.js raw export "NODE_ID" --scale 2 --suffix "_dark"
```

## FigJam Commands

```bash
# List pages
node src/index.js fj list

# Create sticky
node src/index.js fj sticky "Text" -x 100 -y 100

# Create shape
node src/index.js fj shape "Label" -x 200 -y 100

# Connect nodes
node src/index.js fj connect "2:30" "2:34"

# List elements
node src/index.js fj nodes

# Execute JS
node src/index.js fj eval "figma.currentPage.children.length"
```

## Reusable Patterns

### Scale and Center Content

```bash
node src/index.js eval "
const ids = ['1:92', '1:112', '1:134'];  // content group IDs
const frameW = 1920, frameH = 1080;

ids.forEach(id => {
  const n = figma.getNodeById(id);
  if (n) {
    n.rescale(1.2);  // or 0.9 for scale down
    n.x = (frameW - n.width) / 2;
    n.y = (frameH - n.height) / 2;
  }
});
"
```

### Switch Variable Mode (Light/Dark)

```bash
node src/index.js eval "
const node = figma.getNodeById('1:92');

function findModeCollection(n) {
  if (n.boundVariables) {
    for (const [prop, binding] of Object.entries(n.boundVariables)) {
      const b = Array.isArray(binding) ? binding[0] : binding;
      if (b && b.id) {
        try {
          const variable = figma.variables.getVariableById(b.id);
          if (variable) {
            const col = figma.variables.getVariableCollectionById(variable.variableCollectionId);
            if (col && col.modes.length > 1) {
              return { col, modes: col.modes };
            }
          }
        } catch(e) {}
      }
    }
  }
  if (n.children) {
    for (const c of n.children) {
      const found = findModeCollection(c);
      if (found) return found;
    }
  }
  return null;
}

const found = findModeCollection(node);
if (found) {
  const mode = found.modes.find(m => m.name.includes('Light'));  // or 'Dark'
  if (mode) {
    const ids = ['1:92', '1:112', '1:134'];
    ids.forEach(id => {
      const n = figma.getNodeById(id);
      if (n) n.setExplicitVariableModeForCollection(found.col, mode.modeId);
    });
  }
}
"
```

### Rename Nodes

```bash
node src/index.js eval "
const page = figma.currentPage;
page.children.filter(n => n.name.startsWith('Stream-')).forEach((f, i) => {
  f.name = 'session-' + (i + 1);
});
"
```

## Things To Remember

1. **Eval often returns no output** but code still executes. Verify with queries.

2. **Use rescale() not resize()** for scaling. resize() can break layers.

3. **Library variables** cannot be accessed via `getLocalVariableCollections()`. Must find through `boundVariables` on nodes.

4. **Node IDs** are in format `PAGE:NODE` like `1:92`. Get them from query output.

5. **Working directory** must be the repo root containing `src/index.js`.

## File Structure

```
repo-root/
├── src/index.js     # Main CLI, all commands
├── package.json     # npm config
├── README.md        # User docs
└── docs/
    ├── ARCHITECTURE.md   # How it works
    ├── COMMANDS.md       # All commands
    ├── TECHNIQUES.md     # Advanced patterns
    └── CODEX-SESSION.md # This file
```
