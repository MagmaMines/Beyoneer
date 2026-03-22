# BPI — Beyoneer Programming Interface
### Official Developer Reference · IDE · Audited & Corrected

> **Beyoneer IDE** is a full-featured, browser-based IDE. The **BPI** (Beyoneer Programming Interface) is its plugin and scripting API — a collection of powerful namespaces that let you extend, automate, and hook into every part of the IDE from JavaScript.

---

## Table of Contents

1. [What is BPI?](#what-is-bpi)
2. [Quick Start — Your First Plugin](#quick-start--your-first-plugin)
3. [Plugin Anatomy](#plugin-anatomy)
4. [Lifecycle Hooks — `OnThe_Function`](#lifecycle-hooks--onthe_function)
5. [The `BPI` Master Namespace](#the-bpi-master-namespace)
6. [API Reference](#api-reference)
   - [BPI.FS — File System](#bpifs--file-system)
   - [BPI.Editor() — Editor Control](#bpieditor--editor-control)
   - [BPI.Console() — Console Output](#bpiconsole--console-output)
   - [BPI.Execute — Code Execution](#bpiexecute--code-execution)
   - [BPI.Project — Project Management](#bpiproject--project-management)
   - [BPI.Env — Environment Variables](#bpienv--environment-variables)
   - [BPI.Clipboard — Clipboard](#bpiclipboard--clipboard)
   - [BPI.Format — Code Formatting](#bpiformat--code-formatting)
   - [BPI.Search — Find & Replace](#bpisearch--find--replace)
   - [BPI.Diff — File Diff](#bpidiff--file-diff)
   - [BPI.Lint — Code Linting](#bpilint--code-linting)
   - [BPI.Notify — Notifications](#bpinotify--notifications)
   - [BPI.Snippets — Snippet Library](#bpisnippets--snippet-library)
   - [BPI.Http — HTTP Requests](#bpihttp--http-requests)
   - [BPI.UI — Dynamic UI](#bpiui--dynamic-ui)
   - [BPI.Git — Version Control](#bpigit--version-control)
   - [BPI.Task — Background Tasks](#bpitask--background-tasks)
   - [BPI.Event — Event Bus](#bpievent--event-bus)
   - [BPI.KeyMap — Keyboard Shortcuts](#bpikeymap--keyboard-shortcuts)
   - [BPI.Logger — Structured Logging](#bpilogger--structured-logging)
   - [BPI.Config — Plugin Config Storage](#bpiconfig--plugin-config-storage)
   - [BPI.Bench — Performance Benchmarking](#bpibench--performance-benchmarking)
7. [Extended Plugin Globals](#extended-plugin-globals)
   - [BeyoPanel_ — Floating Panels](#beyopanel_--floating-panels)
   - [BeyoTheme_ — Live Theme Control](#beyotheme_--live-theme-control)
   - [BeyoCommand_ — Named Commands](#beyocommand_--named-commands)
   - [BeyoDialog_ — Native Dialogs](#beyodialog_--native-dialogs)
   - [BeyoStatusBar_ — Status Strip](#beyostatusbar_--status-strip)
   - [BeyoWatch_ — File Open Watcher](#beyowatch_--file-open-watcher)
   - [BeyoContextMenu_ — Right-Click Items](#beyocontextmenu_--right-click-items)
   - [BeyoMobile_ — Mobile Helpers](#beyomobile_--mobile-helpers)
   - [BeyoFooterChip_ — Footer Chips](#beyofooterchip_--footer-chips)
   - [BeyoSnippet_ / BeyoInsertSnippet_](#beyosnippet_--beyoinsertsnippet_)
8. [Plugin UI Helpers](#plugin-ui-helpers)
9. [Complete Plugin Examples](#complete-plugin-examples)
10. [Tips & Best Practices](#tips--best-practices)
11. [Getting Help](#getting-help)

---

## What is BPI?

BPI is the scripting backbone of Beyoneer IDE. It exposes a global `window.BPI` object (and individual `window.Beyo*_` globals) that your plugin JavaScript can call to:

- Read and write project files
- Control the CodeMirror 6 editor
- Show toast notifications and dialogs
- Make HTTP requests to external APIs
- Listen for IDE events and create keyboard shortcuts
- Store persistent plugin configuration
- Snapshot and restore editor history (Git-style)
- Benchmark your code's performance

Every BPI function is **available directly in the browser console** of Beyoneer IDE, so you can experiment interactively before writing a plugin.

```js
// Try these in the Beyoneer console right now:
BPI.help()                              // Prints the full API reference table
BPI.version                             // "1.15"
BPI.Notify.success('Hello from BPI!')
BPI.FS.listFiles()                      // List all files in the project
```

---

## Quick Start — Your First Plugin

### Step 1 — Open the Plugin Manager

In Beyoneer IDE, click the **Plugins** button in the toolbar or footer area.

### Step 2 — Create a New Plugin

Click **+ New** and enter a name, for example: `Hello World`.

The IDE generates this starter code automatically:

```js
// Hello World
OnThe_Function('afterBoot', function() {
  console.log('Plugin loaded: Hello World');
  Toast_notification_({ message: 'Hello World is running!', type: 'success' });
});
```

### Step 3 — Save and Activate

Click **Save**, then toggle the **Active** switch on. Reload or click **Test** — your plugin runs on boot.

### Step 4 — Use BPI APIs

```js
OnThe_Function('afterBoot', function() {
  // Show a welcome notification
  BPI.Notify.success('My plugin is live!');

  // Log the current project name
  var info = BPI.Project.info();
  BPI.Console().log('Project: ' + info.name + ' (' + info.fileCount + ' files)');

  // Register a keyboard shortcut
  BPI.KeyMap.bind('Ctrl+Shift+H', function() {
    BPI.Notify.info('Hello from my shortcut!');
  }, 'My Hello shortcut');
});
```

---

## Plugin Anatomy

A plugin is a plain JavaScript string stored in the IDE. It has three parts:

```js
// ── 1. META (optional comments at top) ───────────────────────────
// My Plugin Name
// Description: Does something useful

// ── 2. HOOK REGISTRATION ─────────────────────────────────────────
OnThe_Function('afterBoot', function() {
  // Runs when the IDE finishes loading

  // ── 3. BPI API CALLS ──────────────────────────────────────────
  BPI.Notify.success('Plugin ready!');
  add_footer('My Plugin', function() { runMyFeature(); });
});

// ── HELPER FUNCTIONS (outside the hook) ──────────────────────────
function runMyFeature() {
  var code = BPI.Editor().getValue();
  BPI.Console().log('File has ' + code.length + ' characters.');
}
```

| Part | Required | Purpose |
|---|---|---|
| Meta comments | No | Human-readable description |
| `OnThe_Function(hook, fn)` | Yes | Registers your plugin to an IDE lifecycle event |
| BPI API calls | No | What your plugin actually does |
| Helper functions | No | Supporting logic (declare outside hooks) |

---

## Lifecycle Hooks — `OnThe_Function`

`OnThe_Function(hookName, callback)` is the main way to tie your plugin to IDE events.

```js
OnThe_Function('afterBoot', function() { /* ... */ });
```

### Available Hooks (Verified from Source)

| Hook Name | When It Fires | Callback Arguments |
|---|---|---|
| `afterBoot` | IDE finishes loading & plugins execute — **the main hook** | _(none)_ |
| `onFileOpen` | A file is opened in the editor tab | `(path, content)` |
| `beforeSave` | Just before a project save is committed | `(projectId, projectData)` |
| `onFileSystemChange` | A file is created, deleted, or renamed via BPI.FS | `(action, path)` where action = `'create'` \| `'delete'` \| `'rename'` |

> ⚠️ **Note:** `onSave`, `onRun`, and `onTabChange` do **not** exist in v1.15. Do not use them.

You can register the same plugin to multiple hooks:

```js
OnThe_Function('afterBoot',   onBoot);
OnThe_Function('onFileOpen',  onFileOpen);
OnThe_Function('beforeSave',  onBeforeSave);

function onBoot()                      { BPI.Notify.info('Plugin loaded'); }
function onFileOpen(path, content)     { BPI.Console().log('Opened: ' + path); }
function onBeforeSave(projId, proj)    { BPI.Console().log('Saving project: ' + projId); }
```

---

## The `BPI` Master Namespace

The `BPI` object is the single entry point to all APIs. The lower-level `window.Beyo*_` globals are identical references.

```js
BPI.FS           === window.BeyoFS
BPI.Execute      === window.BeyoExecute_
BPI.Notify       === window.BeyoNotify_
BPI.Editor()     // function call → returns window.BeyoEditor_
BPI.Console()    // function call → returns window.BeyoConsole_
```

Call `BPI.help()` in the console at any time for a live formatted reference table.

---

## API Reference

---

### BPI.FS — File System

Read, write, and manage files in the current project.

> ⚠️ **Corrected method names** — these are the real names from source, not the original docs.

```js
// Read a file (throws if not found)
const content = BPI.FS.readFile('src/index.js');

// Write (create or overwrite) a file
BPI.FS.writeFile('output/result.txt', 'Hello, world!');

// Delete a file
BPI.FS.deleteFile('temp/scratch.js');

// List all files (optionally filter by extension or substring)
const allFiles = BPI.FS.listFiles();           // all files
const jsFiles  = BPI.FS.listFiles('.js');      // only .js files
const srcFiles = BPI.FS.listFiles('src/');     // only files containing 'src/'

// Check if a file exists
if (BPI.FS.fileExists('config.json')) {
  const cfg = JSON.parse(BPI.FS.readFile('config.json'));
}

// Get file metadata { path, size, type, modified }
const info = BPI.FS.getFileInfo('src/app.js');
BPI.Console().log('Size: ' + info.size + ' bytes');

// Copy a file
BPI.FS.copyFile('src/template.js', 'src/myComponent.js');

// Rename / move a file
BPI.FS.renameFile('old-name.js', 'new-name.js');

// Get the current project name as a string
const name = BPI.FS.getProjectName();
```

> 📝 **BPI.FS does not have `readJSON()` or `writeJSON()`.** Use `JSON.parse(BPI.FS.readFile(path))` and `BPI.FS.writeFile(path, JSON.stringify(obj, null, 2))` instead.

---

### BPI.Editor() — Editor Control

Direct access to the CodeMirror 6 editor instance. Note: this is a **function call**, not a property.

> ⚠️ **Corrected method names** — several methods from the original docs did not exist.

```js
const editor = BPI.Editor();

// Get all text in the editor
const code = editor.getValue();

// Replace all editor content (also saves to project)
editor.setValue('// New content\nconsole.log("hi");');

// Insert text at the cursor position (mobile-safe)
editor.insertAtCursor('// inserted here\n');

// Wrap the current selection with before/after strings
editor.wrapSelection('/**', '*/');

// Get the currently selected text
const selection = editor.getSelectedText();

// Get current cursor position — returns character OFFSET (number), not {line,col}
const offset = editor.getCursor();

// Get the current line text and number
const line = editor.getLine(); // { text: '  return x;', number: 42 }

// Get the currently open filename / path
const filename = editor.getActiveFile();  // e.g. 'src/app.js'

// Get the active file's extension
const ext = editor.getActiveExt();        // e.g. 'js'

// Open a file by path
editor.openFile('src/utils.js');

// Save the active file explicitly
editor.saveActiveFile();

// Programmatically create a new project file
editor.createFile('src/newFile.js', '// content');

// Prepend a line to the top of the file
editor.prependLine('// AUTO-GENERATED');

// Append a line to the bottom of the file
editor.appendLine('// end of file');

// Replace all occurrences of a string in the editor
editor.replaceAll('oldFunctionName', 'newFunctionName');

// Convenience FS proxies (same as BPI.FS)
editor.listFiles('.js');
editor.readFile('src/app.js');
editor.writeFile('src/app.js', code);

// Trigger Prettier formatting on the current file
editor.formatCode();
```

> ⚠️ There is no `gotoLine()` method. To jump to a line, use `BPI.Mobile.selectLines(n, n)` or compute the character offset manually.

---

### BPI.Console() — Console Output

Write to the IDE's built-in console panel. This is a **function call**.

```js
const con = BPI.Console();

con.log('Normal message');
con.warn('Something might be wrong');
con.error('Something went wrong!');    // ✅ .error() works (patched in v1.15)
con.err('Also works');                 // legacy alias kept
con.dim('Quiet / secondary output');
con.clear();                           // ✅ clears the console panel (patched in v1.15)
```

> ⚠️ In the original v1.15 release, `.error()` was silently broken (only `.err()` existed). The patched file fixes this. If you are on an unpatched IDE, use `.err()` instead of `.error()`.

---

### BPI.Execute — Code Execution

Run JavaScript code strings, or schedule and benchmark tasks.

```js
// Run a JavaScript string (returns { ok, result, ms } or { ok, error })
const res = BPI.Execute.run('1 + 2 + 3');
// Logs: [BeyoExec] ✓ Done in 0.12ms → 6

// Run with a custom label (appears in console prefix)
BPI.Execute.run('console.log("hi")', { label: 'MyPlugin' });

// Run the entire current file
BPI.Execute.runCurrentFile();

// Run only the selected text in the editor
BPI.Execute.runSelection();

// Schedule a function after a delay (ms) — returns timer ID
const id = BPI.Execute.schedule(function() {
  BPI.Notify.info('5 seconds passed!');
}, 5000);

// Benchmark a function synchronously (n=100 runs by default)
const bench = BPI.Execute.benchmark(function() {
  let x = 0; for (let i = 0; i < 1000; i++) x += i;
}, 100);
// bench = { avg: 0.012, min: 0.009, max: 0.021 }

// Run an async function with a timeout guard (default 5000ms)
const result = await BPI.Execute.runAsync(async function() {
  const r = await fetch('https://api.example.com/data');
  return r.json();
}, 8000);
if (result.ok) BPI.Console().log(JSON.stringify(result.result));
```

---

### BPI.Project — Project Management

```js
// Get info about the currently open project
const info = BPI.Project.info();
// { id: 'proj_123', name: 'My App', fileCount: 5, files: ['index.js', ...] }

// List all projects
const all = BPI.Project.list();
// [{ id, name, fileCount }, ...]

// Switch to another project by name or ID
BPI.Project.switchTo('My Other Project');

// Create a new project with optional file template
const newId = BPI.Project.create('New App', {
  'index.html': { content: '<!DOCTYPE html><html></html>' },
  'app.js':     { content: '// main entry' }
});

// Get statistics for the current project
const stats = BPI.Project.stats();
// { chars: 15200, lines: 430, files: 7 }

// Duplicate the current project
const copyId = BPI.Project.duplicate('My App — Backup');
```

---

### BPI.Env — Environment Variables

Store and retrieve persistent key-value configuration. Data survives page reloads.

```js
// Set a variable
BPI.Env.set('API_KEY', 'sk-abc123');
BPI.Env.set('DEBUG_MODE', true);

// Get a variable (with optional fallback)
const key   = BPI.Env.get('API_KEY', 'not-set');
const debug = BPI.Env.get('DEBUG_MODE', false);

// Check existence
if (BPI.Env.has('API_KEY')) { /* ... */ }

// Delete a key
BPI.Env.delete('OLD_KEY');

// Get all variables as a plain object
const all = BPI.Env.all();

// Get all key names
const keys = BPI.Env.keys();

// Toggle a boolean flag (true → false → true)
const isDebug = BPI.Env.toggle('DEBUG_MODE');

// Print all vars to the IDE console
BPI.Env.dump();

// Clear all env vars
BPI.Env.clear();
```

**Example — API key storage plugin:**
```js
OnThe_Function('afterBoot', function() {
  add_footer('Set API Key', async function() {
    var key = await BPI.Notify.prompt('Enter your API key:', 'sk-...');
    if (key) {
      BPI.Env.set('MY_API_KEY', key);
      BPI.Notify.success('API key saved!');
    }
  });
});
```

---

### BPI.Clipboard — Clipboard

```js
// Copy any text to clipboard
await BPI.Clipboard.copy('Hello, clipboard!');

// Copy the full editor content
await BPI.Clipboard.copyEditor();

// Copy only the selected text
await BPI.Clipboard.copySelection();

// Paste clipboard text into editor at cursor
await BPI.Clipboard.pasteToEditor();

// Read text from clipboard (async)
const text = await BPI.Clipboard.read();

// Copy a specific project file's content
await BPI.Clipboard.copyFile('src/utils.js');
```

---

### BPI.Format — Code Formatting & Transforms

```js
// Format the current file using Prettier (async)
await BPI.Format.formatCurrent();

// Minify a JS string (strips comments & collapses whitespace)
const minified = BPI.Format.minify(code);

// Convert tabs ↔ spaces
const spaced = BPI.Format.tabsToSpaces(code, 2);
const tabbed = BPI.Format.spacesToTabs(code, 2);

// Case conversion
BPI.Format.camelToSnake('myVariableName');  // → "my_variable_name"
BPI.Format.snakeToCamel('my_variable_name'); // → "myVariableName"

// Base64 encode / decode
const encoded = BPI.Format.toBase64('Hello!');
const decoded = BPI.Format.fromBase64(encoded);

// Sort / deduplicate lines
const sorted  = BPI.Format.sortLines(code);
const deduped = BPI.Format.dedupLines(code);

// Count words, lines, characters
const stats = BPI.Format.wordCount(code);
// { words: 120, lines: 45, chars: 980 }
```

---

### BPI.Search — Find & Replace

```js
// Find all occurrences in the current file
const matches = BPI.Search.findAll('console.log');
// [{ index: 42, line: 3, col: 1, match: 'console.log' }, ...]

// Case-sensitive search
const exact = BPI.Search.findAll('myVar', true);

// Count occurrences
const count = BPI.Search.count('TODO');

// Replace all in the editor
const replaced = BPI.Search.replaceAll('var ', 'const ');
// Returns number of replacements made

// Search across all project files
const results = BPI.Search.findInProject('fetchData');
// [{ path: 'src/api.js', line: 12, text: 'async function fetchData() {' }, ...]
```

---

### BPI.Diff — File Diff

```js
// Diff two strings (prints to console, returns stats)
const result = BPI.Diff.diff(oldCode, newCode, 'Before', 'After');
// { added: 2, removed: 1, changed: 3, lines: [...] }

// Diff two project files by path
BPI.Diff.diffFiles('src/app-old.js', 'src/app-new.js');

// Diff the editor's unsaved content vs the saved file on disk
BPI.Diff.diffEditorVsFile('src/app.js');
```

---

### BPI.Lint — Code Linting

```js
// Check syntax of any JS string (no execution)
const check = BPI.Lint.checkSyntax('function hello( { return 1; }');
// { ok: false, error: "Unexpected token '{'" }

// Lint the current file (reports to console, shows toast)
const issues = BPI.Lint.lintCurrent();
// [{ line: 5, msg: 'Use of eval() detected' }, ...]

// Lint all JS/TS files in the project
BPI.Lint.lintProject();
```

**What BeyoLint_ detects:**

- Lines longer than 200 characters
- `eval()` usage
- `console.log` left in code
- `debugger` statements
- `TODO` / `FIXME` / `HACK` comments
- `var` declarations (suggests `let`/`const`)

---

### BPI.Notify — Notifications

> ⚠️ **Bug fix:** In the original IDE, the `duration` parameter to `Toast_notification_` was silently ignored. This is fixed in the patched file. The methods below work correctly with the patch.

```js
// Toast notifications (auto-dismiss after duration ms)
BPI.Notify.success('File saved!');              // default 3000ms
BPI.Notify.error('Something went wrong');       // default 4000ms
BPI.Notify.warn('Are you sure?');               // default 3000ms
BPI.Notify.info('Tip: press Ctrl+S to save');   // default 2500ms

// Custom duration
BPI.Notify.success('Long message!', 6000);

// Vibrate (mobile only, pattern in ms)
BPI.Notify.vibrate([50, 30, 50]);

// Confirm dialog (async — returns true/false)
const confirmed = await BPI.Notify.confirm(
  'Delete File',
  'This will permanently delete <strong>index.js</strong>.'
);
if (confirmed) BPI.FS.deleteFile('index.js');

// Input prompt dialog (async — returns string or null if cancelled)
const name = await BPI.Notify.prompt('New project name:', 'My App');
if (name) BPI.Project.create(name);
```

---

### BPI.Snippets — Snippet Library

```js
// Save a snippet (with optional tags array)
BPI.Snippets.save('fetchHelper', `
async function fetchJSON(url) {
  const res = await fetch(url);
  return res.json();
}`, ['utils', 'async']);

// Get snippet code as a string
const code = BPI.Snippets.get('fetchHelper');

// Insert a snippet at editor cursor
BPI.Snippets.insert('fetchHelper');

// List all saved snippet names
const names = BPI.Snippets.list();

// Search snippets by name, tag, or code content
const results = BPI.Snippets.search('async');

// Print all snippets to console
BPI.Snippets.dump();

// Delete a snippet
BPI.Snippets.delete('fetchHelper');
```

---

### BPI.Http — HTTP Requests

```js
// GET request (auto-parses JSON if response is JSON)
const res = await BPI.Http.get('https://api.github.com/users/octocat');
if (res.ok) BPI.Console().log('Name: ' + res.data.name);
// res = { ok: true, status: 200, data: {...} }

// GET with custom headers
const auth = await BPI.Http.get(
  'https://api.example.com/profile',
  { 'Authorization': 'Bearer ' + BPI.Env.get('TOKEN') }
);

// POST request (body auto-serialized to JSON)
const post = await BPI.Http.post('https://api.example.com/items', {
  title: 'New Item',
  done: false
});
BPI.Console().log('Created: ' + post.data.id);

// Dynamically load an external script (cached — won't double-load)
await BPI.Http.loadScript('https://cdn.example.com/chart.min.js');
// Now window.Chart is available
```

---

### BPI.UI — Dynamic UI

```js
// Create a draggable floating panel
BPI.UI.floatingPanel({
  id:    'myPanel',
  title: 'My Plugin Panel',
  html:  '<p>Hello!</p><button onclick="BPI.Notify.success(\'Hi!\')">Click</button>',
  x:     60,    // initial left (px)
  y:     100,   // initial top (px)
  width: 340    // panel width (px)
});

// Close a panel by id
BPI.UI.closePanel('myPanel');

// Add a status chip to the IDE status bar
BPI.UI.statusChip({
  id:    'myStatus',
  text:  '● Connected',
  color: '#4ade80',
  icon:  'ri-wifi-line'     // RemixIcon class name
});

// Remove a status chip
BPI.UI.removeChip('myStatus');
```

---

### BPI.Git — Version Control

Snapshot, restore, and diff editor content — a lightweight in-session Git.

```js
// Commit current editor content
BPI.Git.commit('Before big refactor');

// List all commits for current file (prints to console)
BPI.Git.log();

// Restore a snapshot by index
BPI.Git.checkout(0);   // first commit
BPI.Git.checkout();    // latest commit (no argument)

// Diff current editor vs the last commit
BPI.Git.diffHead();

// Clear commit history for current file
BPI.Git.reset();

// Show all files that have commits
BPI.Git.status();
```

> Snapshots are stored in `localStorage`. Up to 50 commits per file are kept, and they survive page reloads.

---

### BPI.Task — Background Tasks

```js
// Register a named async task
BPI.Task.register('buildProject', async function() {
  BPI.Console().log('Building...');
  await new Promise(r => setTimeout(r, 2000));
  BPI.Console().log('Done!');
});

// Run it (toasts on start, success, or failure)
await BPI.Task.run('buildProject');

// Check if running
if (BPI.Task.isRunning('buildProject')) {
  BPI.Notify.warn('Already running!');
}

// List all registered task names
BPI.Task.list();

// Remove a task
BPI.Task.unregister('buildProject');
```

---

### BPI.Event — Event Bus

Publish and subscribe to custom IDE-wide events so plugins can talk to each other.

```js
// Subscribe to a custom event (returns subscription id)
const subId = BPI.Event.on('myPlugin:ready', function(payload) {
  BPI.Console().log('Data: ' + JSON.stringify(payload));
});

// Subscribe once (auto-removes after first fire)
BPI.Event.once('myPlugin:ready', function(data) {
  BPI.Notify.success('Got ' + data.count + ' items');
});

// Emit an event with a payload
BPI.Event.emit('myPlugin:ready', { count: 42, items: ['a', 'b', 'c'] });

// Unsubscribe by the id returned from .on()
BPI.Event.off('myPlugin:ready', subId);

// List all active event channel names
BPI.Event.channels();

// Remove all listeners for a channel
BPI.Event.clear('myPlugin:ready');
```

---

### BPI.KeyMap — Keyboard Shortcuts

```js
// Bind a shortcut (combo, handler, description)
BPI.KeyMap.bind('Ctrl+Shift+F', function() {
  BPI.Format.formatCurrent();
}, 'Format current file');

BPI.KeyMap.bind('Ctrl+Shift+L', function() {
  BPI.Lint.lintCurrent();
}, 'Lint current file');

// Unbind a shortcut
BPI.KeyMap.unbind('Ctrl+Shift+F');

// List all registered shortcuts
BPI.KeyMap.list();
```

> Combo strings are case-insensitive and normalized (e.g. `ctrl+shift+f` = `CTRL+SHIFT+F`).

---

### BPI.Logger — Structured Logging

Tagged, levelled logging with a queryable in-memory history (max 1000 entries).

```js
// Log at different levels
BPI.Logger.info('MyPlugin',  'Initialization complete');
BPI.Logger.warn('MyPlugin',  'Config missing — using defaults');
BPI.Logger.error('MyPlugin', 'API call failed: 503');
BPI.Logger.debug('MyPlugin', 'Processing item #42');

// Query log history
const errors = BPI.Logger.query({ level: 'ERROR', limit: 20 });
const myLogs = BPI.Logger.query({ tag: 'MyPlugin', limit: 50 });

// Print summary to console
BPI.Logger.summary();
// [BeyoLogger] Summary — INFO:5 WARN:2 ERROR:1 DEBUG:8 | Total:16

// Export all logs as JSON string
const json = BPI.Logger.export();

// Clear all log history
BPI.Logger.clear();
```

---

### BPI.Config — Plugin Config Storage

Persistent per-plugin config in `localStorage` with automatic namespacing. Each plugin gets its own isolated key `bpi_cfg_<pluginId>`.

```js
// Save config (merges into existing)
BPI.Config.set('MyPlugin', { theme: 'dark', fontSize: 14, autoSave: true });

// Read the full config object
const cfg = BPI.Config.get('MyPlugin');
// { theme: 'dark', fontSize: 14, autoSave: true }

// Read a single key with fallback
const theme = BPI.Config.read('MyPlugin', 'theme', 'light');

// Write a single key (merges, doesn't overwrite other keys)
BPI.Config.write('MyPlugin', 'fontSize', 16);

// Reset (delete) all config for a plugin
BPI.Config.reset('MyPlugin');

// List all plugins that have saved config
BPI.Config.list();
```

---

### BPI.Bench — Performance Benchmarking

```js
// Named timer
BPI.Bench.start('myOperation');
// ... do work ...
const ms = BPI.Bench.end('myOperation');
// Logs: [BeyoBench] ⏱ myOperation: 12.345ms

// Benchmark a sync function (n iterations)
BPI.Bench.run('sort', function() {
  [5, 3, 1, 4, 2].sort(function(a, b) { return a - b; });
}, 1000);
// Logs: [BeyoBench] 🔁 sort × 1000: avg 0.0021ms/iter

// Benchmark an async function
await BPI.Bench.runAsync('fetch', async function() {
  await fetch('/api/ping');
}, 10);

// Report JS heap memory (Chrome only)
const mem = BPI.Bench.memory();
// { usedMB: 14.23, limitMB: 512.0 }
```

---

## Extended Plugin Globals

These are additional globals available inside all plugin code, beyond the `BPI.*` namespace.

---

### BeyoPanel_ — Floating Panels

A simpler floating panel API (toggles on/off with the same call).

```js
// Show a floating panel — call again with same id to close it
BeyoPanel_({
  id:       'myDevPanel',
  title:    'Dev Tools',
  html:     '<p>Custom panel content here.</p>',
  position: 'bottom-right',   // 'bottom-right' | 'bottom-left' | 'top-right' | 'top-left' | 'center'
  width:    320,
  height:   240,
  icon:     'ri-tools-line'
});
```

---

### BeyoTheme_ — Live Theme Control

Override any IDE CSS variable instantly.

```js
BeyoTheme_({
  primary: '#e879f9',   // accent color
  bg:      '#0d0221',   // main background
  panel:   '#160420',   // sidebar/panel background
  editor:  '#1a0535',   // editor background
  border:  '#3b0764',   // border color
  success: '#4ade80',
  error:   '#f87171',
  font:    16           // editor font size in px (or '16px' string)
});
```

---

### BeyoCommand_ — Named Commands

Register a named command with an optional keyboard shortcut.

```js
BeyoCommand_({
  name:     'My Command',
  shortcut: 'Ctrl+Shift+M',    // optional
  icon:     'ri-command-line', // optional
  action:   function() {
    BPI.Notify.success('Command executed!');
  }
});
```

---

### BeyoDialog_ — Native Dialogs

Show the IDE's native modal dialog with custom content and action buttons.

```js
BeyoDialog_({
  title:   'Confirm Action',
  icon:    'ri-question-line',
  content: '<p style="color:var(--text-muted)">Are you sure?</p>',
  actions: [
    { label: 'Cancel',  action: 'Dlg.close()' },
    { label: 'Confirm', action: 'Dlg.close(); doMyAction()', primary: true }
  ]
});
```

---

### BeyoStatusBar_ — Status Strip

A thin strip pinned to the bottom of the screen (not the same as `BPI.UI.statusChip`).

```js
// Show / update a status bar strip
BeyoStatusBar_({
  id:    'myStatus',         // optional — allows updating existing strip
  text:  'Build: Running…',
  color: '#fbbf24',
  icon:  'ri-loader-4-line'
});

// To remove it, set empty text or call without id to replace
BeyoStatusBar_({ id: 'myStatus', text: '' });
```

---

### BeyoWatch_ — File Open Watcher

Fires a callback whenever a matching file is opened in the editor.

```js
// Watch for a specific file path
BeyoWatch_('src/app.js', function(path, content) {
  BPI.Console().log('app.js opened — ' + content.length + ' chars');
});

// Watch for a file extension (starts with '.')
BeyoWatch_('.py', function(path, content) {
  BPI.Notify.info('Python file opened: ' + path);
});
```

---

### BeyoContextMenu_ — Right-Click Items

Add a custom item to the file tree right-click context menu.

```js
BeyoContextMenu_({
  label:  'Minify This File',
  icon:   'ri-scissors-line',
  filter: ['js', 'ts'],        // optional — only show for these extensions
  action: function(path) {
    var code = BPI.FS.readFile(path);
    BPI.FS.writeFile(path, BPI.Format.minify(code));
    BPI.Notify.success('Minified: ' + path);
  }
});
```

---

### BeyoMobile_ — Mobile Helpers

Utilities for touch devices.

```js
// Check environment
BeyoMobile_.isTouch();    // true on touch devices
BeyoMobile_.isMobile();   // true if screen width ≤ 768px

// Add a swipe gesture handler
// direction: 'left' | 'right' | 'up' | 'down'
BeyoMobile_.addSwipeAction('right', function(dx) {
  // dx = pixel distance of swipe
  document.getElementById('sidebar').classList.add('show');
});

// Add a button to the floating mobile dock
BeyoMobile_.addDockButton({
  id:     'myDockBtn',          // prevents duplicate on re-run
  icon:   'ri-magic-line',
  label:  'My Action',
  color:  '#a78bfa',
  action: function() { BPI.Notify.success('Dock button pressed!'); }
});

// Trigger haptic feedback
BeyoMobile_.haptic('light');    // 'light' | 'medium' | 'heavy'

// Set editor font size
BeyoMobile_.fontScale(16);

// Open / close sidebar
BeyoMobile_.openSidebar();
BeyoMobile_.closeSidebar();

// Programmatically select a line range in the editor
BeyoMobile_.selectLines(5, 12);  // selects lines 5 through 12

// Show a line-picker dialog (mobile-friendly)
BeyoMobile_.showLinePicker({
  title:    'Pick a line',
  multi:    true,              // allow selecting a range (default: true)
  onSelect: function(from, to) {
    BPI.Console().log('Selected lines ' + from + '-' + to);
  }
});
```

---

### BeyoFooterChip_ — Footer Chips

Full-config version of `add_footer` with an explicit remove API.

```js
// Add a footer chip
BeyoFooterChip_({
  id:      'myChip',
  name:    'My Tool',
  icon:    'ri-magic-line',
  color:   '#a78bfa',
  tooltip: 'Run My Tool',
  action:  function() { BPI.Notify.success('My tool ran!'); }
});

// Remove a footer chip by id
BeyoFooterChip_({ id: 'myChip', remove: true });
```

---

### BeyoSnippet_ / BeyoInsertSnippet_

Register and insert editor autocomplete snippets.

```js
// Register a snippet with a trigger keyword
BeyoSnippet_({
  trigger:     'fetchjson',
  description: 'Async fetch helper',
  lang:        'js',
  code:        'async function fetchJSON(url) {\n  const r = await fetch(url);\n  return r.json();\n}'
});

// Insert a registered snippet at cursor
BeyoInsertSnippet_('fetchjson');
```

---

## Plugin UI Helpers

These global helper functions are the standard way to inject UI into the IDE.

### `add_footer(name, fn, opts?)`

Adds a clickable chip to the plugin footer bar. Clicking it calls `fn`.

```js
add_footer('Format', function() { BPI.Format.formatCurrent(); });
add_footer('Lint',   function() { BPI.Lint.lintCurrent(); }, {
  icon:    'ri-bug-line',
  color:   '#f87171',
  tooltip: 'Lint this file',
  id:      'myLintChip'   // optional stable id to prevent duplicates
});
```

### `add_header(name, fn, opts?)`

Adds a button to the IDE header toolbar.

```js
add_header('Deploy', function() { deploy(); }, {
  icon:    'ri-upload-cloud-line',
  tooltip: 'Deploy project',
  color:   '#4ade80'
});
```

### `Toast_notification_({ message, type, duration? })`

Shows a toast. `type` = `'success'` | `'error'` | `'warn'` | `'info'`.

```js
Toast_notification_({ message: 'Done!', type: 'success' });
Toast_notification_({ message: 'Watch out', type: 'warn', duration: 5000 });
```

---

## Complete Plugin Examples

### Example 1 — Auto-Save Git Snapshot

```js
// Auto Git Snapshot — commits the editor every 5 minutes
OnThe_Function('afterBoot', function() {
  BPI.Logger.info('AutoSnapshot', 'Plugin started');

  setInterval(function() {
    var file = BPI.Editor().getActiveFile();  // ✅ correct method name
    if (!file) return;
    BPI.Git.commit('auto @ ' + new Date().toLocaleTimeString());
    BPI.Logger.info('AutoSnapshot', 'Committed: ' + file);
  }, 5 * 60 * 1000);

  BPI.Notify.info('Auto-snapshot active (every 5 min)');
});
```

---

### Example 2 — Live Word Count Status Chip

```js
// Live word/line count in IDE status bar
OnThe_Function('afterBoot', function() {
  function update() {
    var code = BPI.Editor() && BPI.Editor().getValue() || '';
    var s = BPI.Format.wordCount(code);
    BPI.UI.statusChip({
      id:    'wc',
      text:  s.lines + 'L / ' + s.words + 'W',
      color: '#a78bfa',
      icon:  'ri-text'
    });
  }
  update();
  setInterval(update, 3000);
});
```

---

### Example 3 — GitHub README Fetcher

```js
// Fetch any GitHub repo README into a project file
OnThe_Function('afterBoot', function() {
  add_footer('Fetch README', async function() {
    var repo = await BPI.Notify.prompt('GitHub repo (owner/name):', 'octocat/Hello-World');
    if (!repo) return;
    BPI.Notify.info('Fetching…');
    var url = 'https://raw.githubusercontent.com/' + repo + '/HEAD/README.md';
    var res = await BPI.Http.get(url);
    if (!res.ok) { BPI.Notify.error('Fetch failed: ' + res.status); return; }
    BPI.FS.writeFile('fetched-README.md', res.data);
    BPI.Notify.success('Saved as fetched-README.md');
  });
});
```

---

### Example 4 — Keyboard Shortcut Suite

```js
// Register useful shortcuts as a bundle
OnThe_Function('afterBoot', function() {
  BPI.KeyMap.bind('Ctrl+Shift+F', function() { BPI.Format.formatCurrent(); }, 'Format file');
  BPI.KeyMap.bind('Ctrl+Shift+L', function() { BPI.Lint.lintCurrent(); },     'Lint file');
  BPI.KeyMap.bind('Ctrl+Shift+S', function() { BPI.Git.commit('quick-save'); },'Git snapshot');
  BPI.KeyMap.bind('Ctrl+Shift+W', function() {
    var s = BPI.Format.wordCount(BPI.Editor().getValue());
    BPI.Notify.info(s.words + ' words, ' + s.lines + ' lines');
  }, 'Word count');

  BPI.Notify.success('Dev shortcuts registered (Ctrl+Shift+F/L/S/W)');
});
```

---

### Example 5 — Persistent Config Plugin

```js
// Smart Formatter — remembers indent size across sessions
var PLUGIN_ID = 'SmartFormatter';

OnThe_Function('afterBoot', function() {
  var defaults = { indentSize: 2 };
  var cfg = Object.assign({}, defaults, BPI.Config.get(PLUGIN_ID));

  add_footer('Format (Smart)', function() {
    BPI.Format.formatCurrent();
    BPI.Logger.info(PLUGIN_ID, 'Formatted, indent=' + cfg.indentSize);
  });

  add_footer('Formatter Settings', async function() {
    var size = await BPI.Notify.prompt('Indent size (2 or 4):', String(cfg.indentSize));
    if (!size) return;
    cfg.indentSize = parseInt(size) || 2;
    BPI.Config.set(PLUGIN_ID, cfg);
    BPI.Notify.success('Settings saved!');
  });
});
```

---

### Example 6 — Event Bus Between Two Plugins

**Plugin A — Data Publisher:**
```js
OnThe_Function('afterBoot', function() {
  add_footer('Emit File Info', function() {
    var data = { ts: Date.now(), file: BPI.Editor().getActiveFile() };
    BPI.Event.emit('sharedTools:fileInfo', data);
    BPI.Notify.info('Data emitted!');
  });
});
```

**Plugin B — Data Subscriber:**
```js
OnThe_Function('afterBoot', function() {
  BPI.Event.on('sharedTools:fileInfo', function(data) {
    BPI.Console().log('Received: ' + JSON.stringify(data));
    BPI.Notify.success('File: ' + data.file);
  });
});
```

---

### Example 7 — File Watcher + Theme Switcher

```js
// Apply Synthwave theme when a .css file is opened
OnThe_Function('afterBoot', function() {
  BeyoWatch_('.css', function(path) {
    BeyoTheme_({ primary: '#e879f9', bg: '#0d0221', panel: '#160420' });
    BPI.Notify.info('Synthwave theme — editing: ' + path);
  });
});
```

---

### Example 8 — Custom Context Menu Item

```js
// Add "Minify" to the right-click menu for JS files
OnThe_Function('afterBoot', function() {
  BeyoContextMenu_({
    label:  'Minify File',
    icon:   'ri-scissors-line',
    filter: ['js', 'ts'],
    action: function(path) {
      var code = BPI.FS.readFile(path);
      BPI.FS.writeFile(path, BPI.Format.minify(code));
      BPI.Notify.success('Minified: ' + path);
    }
  });
});
```

---

## Tips & Best Practices

**1. Always guard against a missing editor**
```js
var editor = BPI.Editor();
if (!editor) return;
var code = editor.getValue();
```

**2. Use `BPI.Logger` instead of `console.log` in production plugins**
```js
// Bad — appears in browser dev console, not the IDE console
console.log('my debug info');

// Good — queryable, tagged, appears in the IDE console panel
BPI.Logger.debug('MyPlugin', 'my debug info');
```

**3. Use `.error()` not `.err()` for BPI-style error output**
```js
// Both work after the v1.15 patch — prefer .error() for readability
BPI.Console().error('Something failed');
```

**4. Namespace your events to avoid collisions**
```js
// Bad — 'ready' could clash with any other plugin
BPI.Event.emit('ready', data);

// Good — unique to your plugin
BPI.Event.emit('MyPlugin:ready', data);
```

**5. Store secrets in `BPI.Env`, not hardcoded in plugin code**
```js
// Bad — anyone who opens the plugin editor can see this
const token = 'ghp_abc123xyz';

// Good — stored separately from plugin code
const token = BPI.Env.get('GITHUB_TOKEN');
```

**6. Use the correct method names for BPI.FS**

| ❌ Wrong (from old docs) | ✅ Correct |
|---|---|
| `BPI.FS.list()` | `BPI.FS.listFiles()` |
| `BPI.FS.exists(path)` | `BPI.FS.fileExists(path)` |
| `BPI.FS.copy(a,b)` | `BPI.FS.copyFile(a,b)` |
| `BPI.FS.rename(a,b)` | `BPI.FS.renameFile(a,b)` |
| `BPI.FS.delete(p)` | `BPI.FS.deleteFile(p)` |
| `BPI.FS.size(p)` | `BPI.FS.getFileInfo(p).size` |

**7. Export your plugins for backup**

In the Plugin Manager, use the **Export** button to download as `.json`. Use **Import** to share between devices or teammates.

**8. Use `BPI.Bench` when your plugin does heavy work**
```js
BPI.Bench.start('myHeavyOp');
// ... expensive work ...
BPI.Bench.end('myHeavyOp');  // automatically logs timing
```

---

## Changelog — v1.15 Patch Notes

These bugs were found during a full source audit and fixed in the patched HTML:

| # | Bug | Impact | Fix |
|---|---|---|---|
| 1 | `BeyoConsole_.error()` silently failed — method did not exist | **Critical** — every BPI module's error logs were swallowed | Added `.error` as proper alias |
| 2 | `Toast_notification_({ duration })` parameter was silently ignored | Medium — custom durations had no effect | Passed `duration` through to `Toast.show()` |
| 3 | `BeyoConsole_.clear()` was documented but missing | Low | Added `.clear()` method |

---

## Getting Help

| Resource | Link |
|---|---|
| 📧 Email Support | [Help.magmamine@gmail.com](mailto:Help.magmamine@gmail.com) |
| 🌐 Official Website | [beyoneer.xyz](https://beyoneer.xyz) |
| 📟 In-IDE Reference | Run `BPI.help()` in the Beyoneer console |
| 💻 Terminal shortcut | Type `bpi help` in the IDE terminal |

---

*BPI Documentation · Beyoneer IDE· By **MagmaMinesTeam***  
*© MagmaMinesTeam — [Help.magmamine@gmail.com](mailto:Help.magmamine@gmail.com) — [beyoneer.xyz](https://beyoneer.xyz)*
