# BPI — Beyoneer Programming Interface
### Official Developer Reference · v1.15 · Audited & Corrected

> **Beyoneer IDE** is a full-featured, browser-based code editor. The **BPI** (Beyoneer Programming Interface) is its plugin and scripting API — a collection of powerful namespaces that let you extend, automate, and hook into every part of the IDE from JavaScript.

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
   - [BPI.Theme — IDE Theme Control ★ New](#bpitheme--ide-theme-control-)
   - [BPI.Color — Color Utilities ★ New](#bpicolor--color-utilities-)
   - [BPI.Storage — Enhanced Storage ★ New](#bpistorage--enhanced-storage-)
   - [BPI.Worker — Web Worker Pool ★ New](#bpiworker--web-worker-pool-)
   - [BPI.Template — Code Template Manager ★ New](#bpitemplate--code-template-manager-)
   - [BPI.I18n — Internationalization ★ New](#bpii18n--internationalization-)
   - [BPI.Execute — Extended Methods ★ Updated](#bpiexecute--extended-methods-)
   - [BPI.FS — Extended Methods ★ Updated](#bpifs--extended-methods-)
   - [BPI.Http — Extended Methods ★ Updated](#bpihttp--extended-methods-)
   - [BPI.Format — Extended Methods ★ Updated](#bpiformat--extended-methods-)
   - [BPI.UI — Extended Methods ★ Updated](#bpiui--extended-methods-)
   - [BPI.Notify — Extended Methods ★ Updated](#bpinotify--extended-methods-)
   - [BPI.Logger — Extended Methods ★ Updated](#bpilogger--extended-methods-)
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
// ── New v1.15 modules ──
BPI.Theme        === window.BeyoTheme_
BPI.Color        === window.BeyoColor_
BPI.Storage      === window.BeyoStorage_
BPI.Worker       === window.BeyoWorker_
BPI.Template     === window.BeyoTemplate_
BPI.I18n         === window.BeyoI18n_
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

// Read a JSON file and parse it automatically
const cfg = BPI.FS.readJson('config.json', {});   // 2nd arg = fallback if missing

// Write an object as a formatted JSON file
BPI.FS.writeJson('data/result.json', { ts: Date.now(), ok: true });

// Append content to a file (creates file if missing)
BPI.FS.appendFile('logs/debug.log', new Date().toISOString() + ' — event\n');

// Move / rename a file (with hook notification)
BPI.FS.moveFile('src/old-name.js', 'src/new-name.js');

// Get word / line / char stats for a file
const stats = BPI.FS.getStats('src/app.js');
// { path, lines, words, chars, type }

// Read ALL project files into a { path: content } map
const all = BPI.FS.readAll();           // every file
const jsAll = BPI.FS.readAll('.js');    // only .js files

// List files filtered by extension array
const webFiles = BPI.FS.listByExt(['html', 'css', 'js']);

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

// Run a project file by path
BPI.Execute.runFile('scripts/build.js');

// Start a repeating interval — returns a stop() function
const stop = BPI.Execute.interval(function() {
  BPI.UI.statusChip({ id: 'clock', text: new Date().toLocaleTimeString(), color: '#a78bfa' });
}, 1000, 'clock-label');
// Call stop() to cancel

// Retry an async function up to N times (retries, delayMs)
const r = await BPI.Execute.retry(async () => {
  const res = await BPI.Http.get('https://api.example.com/data');
  if (!res.ok) throw new Error('HTTP ' + res.status);
  return res.data;
}, 3, 500, 'fetch-retry');

// Debounce — returns a wrapper that fires only after a pause
const debouncedLint = BPI.Execute.debounce(() => BPI.Lint.lintCurrent(), 500);

// Throttle — returns a wrapper that fires at most once per interval
const throttledSave = BPI.Execute.throttle(() => BPI.Git.commit('auto'), 2000);
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

// ── Extended methods (added v1.15) ──────────────────────────────────────

// Convert to PascalCase / kebab-case
BPI.Format.toPascalCase('my component name');  // → "MyComponentName"
BPI.Format.toKebabCase('myVariableName');       // → "my-variable-name"

// HTML entity encode / decode
const safe = BPI.Format.htmlEncode('<script>alert("xss")</script>');
const back = BPI.Format.htmlDecode(safe);

// Truncate a string with ellipsis
BPI.Format.truncate('A very long description text...', 40);

// Strip all HTML tags from a string
BPI.Format.stripHtml('<p>Hello <b>world</b></p>');  // → "Hello world"

// Wrap each line with a prefix/suffix
BPI.Format.wrapLines(code, '// ', '');  // comment out every line

// Format an array of objects as a console table string
const table = BPI.Format.toTable([
  { file: 'app.js', lines: 120 },
  { file: 'main.css', lines: 88 }
]);
BPI.Console().log(table);
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

// ── Extended methods (added v1.15) ──────────────────────────────────────

// Live progress bar toast (call repeatedly; auto-dismisses at 100%)
BPI.Notify.progress(0,  'Building…');
BPI.Notify.progress(50, 'Building…');
BPI.Notify.progress(100,'Building…');  // auto-hides after ~1.2s

// Request browser permission and send a system notification
const sent = await BPI.Notify.desktop('Build Complete', 'All files compiled successfully.');

// Queue multiple toasts with a delay between each
BPI.Notify.queue([
  { text: 'Step 1: Linting…',    type: 'info'    },
  { text: 'Step 2: Formatting…', type: 'info'    },
  { text: '✅ All done!',         type: 'success' }
], 900); // 900ms gap between each
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

// ── Extended methods (added v1.15) ──────────────────────────────────────

// PUT request
const put = await BPI.Http.put('https://api.example.com/items/1', { done: true });

// PATCH request
const patch = await BPI.Http.patch('https://api.example.com/items/1', { title: 'Updated' });

// DELETE request
const del = await BPI.Http.delete('https://api.example.com/items/1');
BPI.Console().log('Deleted: ' + del.ok);

// Dynamically load a CSS stylesheet (id prevents double-loading)
await BPI.Http.loadCSS('https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css', 'animate');

// Shorthand JSON GET — returns parsed object or null on failure
const user = await BPI.Http.getJSON('https://api.github.com/users/octocat');
if (user) BPI.Console().log('Login: ' + user.login);
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

// ── Extended methods (added v1.15) ──────────────────────────────────────

// Show a tooltip anchored below any element (auto-removes after duration)
BPI.UI.tooltip('myButtonId', 'Click to format your file', 2500);

// Build and show a right-click context menu at any coordinate
BPI.UI.contextMenu({
  x: 200, y: 300,
  items: [
    { label: 'Format File', icon: 'ri-magic-line',   action: () => BPI.Format.formatCurrent() },
    { label: 'Lint File',   icon: 'ri-bug-line',     action: () => BPI.Lint.lintCurrent()     },
    'sep',
    { label: 'Word Count',  icon: 'ri-text',         action: () => {
      const s = BPI.Format.wordCount(BPI.Editor().getValue());
      BPI.Notify.info(s.lines + ' lines, ' + s.words + ' words');
    }}
  ]
});

// Show a numeric badge counter on a status chip
BPI.UI.statusChip({ id: 'errors', text: 'Errors', color: '#f87171', icon: 'ri-error-warning-line' });
BPI.UI.badgeCount('errors', 5);   // shows "5" badge; set 0 to remove

// Flash a glow highlight on any element by id
BPI.UI.highlight('pluginFooterBar', '#3b82f6', 800);

// Show / hide a full-screen loading overlay
BPI.UI.loadingOverlay(true, 'Compiling…');
// ... async work ...
BPI.UI.loadingOverlay(false);
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

// ── Extended methods (added v1.15) ──────────────────────────────────────

// Log an array of objects as a formatted console table
BPI.Logger.table('FileAudit', [
  { file: 'index.js', lines: 120, size: '4.2kb' },
  { file: 'app.css',  lines:  88, size: '2.1kb' },
]);

// Group related log entries under a named label
BPI.Logger.groupStart('Build', 'transpile');
BPI.Logger.groupAdd('Build', 'transpile', 'Compiling TypeScript…');
BPI.Logger.groupAdd('Build', 'transpile', 'Bundling modules…');
const entries = BPI.Logger.groupEnd('Build', 'transpile');
// entries = ['Compiling TypeScript…', 'Bundling modules…']
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

---

### BPI.Theme — IDE Theme Control ★ New

Read, modify, save and restore the IDE's CSS variable theme from plugin code.

```js
// Set a single CSS variable instantly
BPI.Theme.setVar('--primary', '#e879f9');
BPI.Theme.setVar('--bg-editor', '#1a0535');

// Get a variable's current value
const accent = BPI.Theme.getVar('--primary');   // e.g. '#3b82f6'

// Snapshot the entire current theme under a name
BPI.Theme.save('myTheme');

// Apply a saved theme (restores all its variables)
BPI.Theme.apply('myTheme');

// List all saved theme names
const themes = BPI.Theme.list();   // ['myTheme', 'synthwave', ...]

// Delete a saved theme
BPI.Theme.delete('myTheme');

// Bulk-patch multiple variables at once (without saving)
BPI.Theme.patch({
  '--primary':    '#e879f9',
  '--bg-dark':    '#0d0221',
  '--bg-panel':   '#160420',
  '--bg-editor':  '#1a0535',
  '--border':     '#3b0764'
});

// Get all current variable values as a plain object
const current = BPI.Theme.current();

// Reset all CSS variables back to Beyoneer defaults
BPI.Theme.reset();

// Check if the IDE is in dark mode
const dark = BPI.Theme.isDark();   // always true in stock Beyoneer

// Get the default theme map (for reference)
const defaults = BPI.Theme.defaults();
```

**Example — Synthwave theme plugin:**
```js
OnThe_Function('afterBoot', function() {
  add_footer('Synthwave', function() {
    BPI.Theme.patch({
      '--primary':   '#e879f9',
      '--bg-dark':   '#0d0221',
      '--bg-panel':  '#160420',
      '--bg-editor': '#1a0535',
      '--border':    '#3b0764',
      '--success':   '#4ade80'
    });
    BPI.Notify.success('🌃 Synthwave theme applied!');
  }, { icon: 'ri-palette-line', color: '#e879f9' });

  add_footer('Reset Theme', function() {
    BPI.Theme.reset();
    BPI.Notify.info('Theme reset to defaults');
  }, { icon: 'ri-refresh-line' });
});
```

> `BPI.Theme` is the new programmatic API. The `BeyoTheme_({…})` global (see Extended Globals) remains available for legacy plugins.

---

### BPI.Color — Color Utilities ★ New

A full color math library for generating palettes, checking accessibility, and converting formats.

```js
// Parse hex → { r, g, b }
const rgb = BPI.Color.hexToRgb('#3b82f6');   // { r: 59, g: 130, b: 246 }

// { r, g, b } → hex string
const hex = BPI.Color.rgbToHex(59, 130, 246); // '#3b82f6'

// Blend two colors by a 0–1 ratio (0 = full A, 1 = full B)
const mid = BPI.Color.mix('#3b82f6', '#e879f9', 0.5);

// Lighten / darken by a 0–1 amount
const lighter = BPI.Color.lighten('#3b82f6', 0.3);
const darker  = BPI.Color.darken('#3b82f6', 0.3);

// WCAG relative luminance
const lum = BPI.Color.luminance('#3b82f6');   // 0–1 float

// WCAG contrast ratio between two colors (higher = more readable)
const ratio = BPI.Color.contrast('#3b82f6', '#09090b');  // e.g. 5.74

// Check AA (4.5:1) or AAA (7:1) accessibility
const ok = BPI.Color.isAccessible('#3b82f6', '#09090b');       // AA check
const aaa = BPI.Color.isAccessible('#fff', '#000', 'AAA');     // AAA check

// Return the best readable color (black or white) for text on a background
const text = BPI.Color.readable('#3b82f6');   // '#ffffff' or '#000000'

// Convert hex to CSS rgba string
const rgba = BPI.Color.toRgba('#3b82f6', 0.5);  // 'rgba(59,130,246,0.5)'

// Invert a color
const inv = BPI.Color.invert('#3b82f6');  // '#c47d09'

// Generate a random hex color
const rand = BPI.Color.randomHex();   // e.g. '#a3f291'
```

**Example — CSS palette generator:**
```js
OnThe_Function('afterBoot', function() {
  add_footer('Palette', async function() {
    var hex = await BPI.Notify.prompt('Base color (hex):', '#3b82f6');
    if (!hex) return;
    var palette = [0.4, 0.2, 0, -0.2, -0.4].map(function(s) {
      return s >= 0 ? BPI.Color.lighten(hex, s) : BPI.Color.darken(hex, -s);
    });
    var css = ':root {\n' + palette.map(function(c, i) {
      return '  --color-' + (i + 1) + ': ' + c + '; /* ' + BPI.Color.contrast(c, '#fff').toFixed(1) + ':1 */'
    }).join('\n') + '\n}';
    BPI.FS.writeFile('palette.css', css);
    BPI.Notify.success('palette.css written (' + palette.length + ' shades)');
  }, { icon: 'ri-palette-line' });
});
```

---

### BPI.Storage — Enhanced Storage ★ New

Three-layer persistent storage: **IndexedDB** (async, large data), **sessionStorage** (tab-scoped), and **localStorage** (persistent, lightweight).

```js
// ── IndexedDB (async, survives reloads, good for large data) ──────────

await BPI.Storage.setIDB('myPlugin:cache', { items: [1,2,3], ts: Date.now() });

const data = await BPI.Storage.getIDB('myPlugin:cache');
// returns the stored value, or null if the key doesn't exist

await BPI.Storage.delIDB('myPlugin:cache');

const keys = await BPI.Storage.keysIDB();   // all IDB keys

await BPI.Storage.clearIDB();               // ⚠️ wipes entire IDB store

// ── Session Storage (cleared when the tab closes) ─────────────────────

BPI.Storage.session.set('draft', { code: '…', ts: Date.now() });
const draft = BPI.Storage.session.get('draft', null);  // 2nd arg = fallback
BPI.Storage.session.del('draft');
const sessionKeys = BPI.Storage.session.keys();
const hasDraft = BPI.Storage.session.has('draft');

// ── Local Storage (persists across reloads) ───────────────────────────

BPI.Storage.local.set('myPlugin:prefs', { theme: 'dark', fontSize: 14 });
const prefs = BPI.Storage.local.get('myPlugin:prefs', {});
BPI.Storage.local.del('myPlugin:prefs');
BPI.Storage.local.clear('myPlugin:');   // clear only keys with prefix
const localKeys = BPI.Storage.local.keys();

// ── Bulk export / import ──────────────────────────────────────────────

// Export all localStorage entries (or filtered by prefix) to JSON string
const json = BPI.Storage.exportLocal('bpi');           // only 'bpi*' keys
BPI.FS.writeFile('backup/storage.json', json);

// Import a previously exported JSON string back into localStorage
BPI.Storage.importLocal(BPI.FS.readFile('backup/storage.json'));
```

> **Tip:** Prefer `BPI.Storage.setIDB` for large arrays or binary-style data. Use `BPI.Storage.local` for small per-plugin config (same as `BPI.Config`, but without namespacing).

---

### BPI.Worker — Web Worker Pool ★ New

Spawn, communicate with, and terminate Web Workers to run expensive code off the main thread.

```js
// ── Quick one-shot: run a function in a worker and get result ─────────
const sum = await BPI.Worker.run(function(data) {
  // This executes in a separate thread
  let s = 0;
  for (let i = 0; i < data.n; i++) s += i;
  return s;
}, { n: 1_000_000 });
BPI.Console().log('Sum: ' + sum);

// ── Persistent worker: spawn → post → listen → terminate ─────────────

// Spawn from an inline function
BPI.Worker.spawn('parser', function(data) {
  // Each message.data triggers this; return value is posted back
  return data.split('\n').length;
});

// Listen for messages from the worker
BPI.Worker.onMessage('parser', function(lineCount) {
  BPI.Notify.info('Lines counted: ' + lineCount);
});

// Post data to the worker
BPI.Worker.post('parser', BPI.Editor().getValue());

// Terminate when done
BPI.Worker.terminate('parser');

// Terminate all active workers at once
BPI.Worker.terminateAll();

// List active worker IDs
const ids = BPI.Worker.list();   // ['parser', ...]
```

> **Browser note:** Workers are created via `Blob` URLs. They have no access to `window`, `document`, or BPI — only standard Web Worker APIs (`self`, `fetch`, `setTimeout`, etc.).

---

### BPI.Template — Code Template Manager ★ New

Save, retrieve, and apply reusable code templates across sessions. Templates are stored in `localStorage` and survive page reloads.

```js
// Save a template (name, code, optional metadata)
BPI.Template.save('fetchHelper', `
async function fetchJSON(url) {
  const res = await fetch(url);
  if (!res.ok) throw new Error(res.status);
  return res.json();
}`, { lang: 'js', tags: ['async', 'util'], desc: 'Fetch JSON with error handling' });

// Get the code string of a template
const code = BPI.Template.get('fetchHelper');

// Get full metadata { code, lang, tags, desc, saved }
const meta = BPI.Template.getMeta('fetchHelper');

// Apply to editor (replaces all content)
BPI.Template.apply('fetchHelper');

// Insert at cursor (does not replace existing content)
BPI.Template.insert('fetchHelper');

// Save the current editor content as a template
BPI.Template.saveFromEditor('myBoilerplate', { lang: 'js', tags: ['starter'] });

// List all template names (optionally filter by lang or tag)
const allNames = BPI.Template.list();
const jsNames  = BPI.Template.list({ lang: 'js' });
const utilNames = BPI.Template.list({ tag: 'util' });

// Rename / delete templates
BPI.Template.rename('fetchHelper', 'jsonFetcher');
BPI.Template.delete('jsonFetcher');

// Export all templates as a JSON string
const json = BPI.Template.export();
BPI.FS.writeFile('templates/backup.json', json);

// Import templates from JSON (merges with existing)
BPI.Template.import(BPI.FS.readFile('templates/backup.json'));

// Clear all templates
BPI.Template.clear();
```

**Example — Template picker in the footer:**
```js
OnThe_Function('afterBoot', async function() {
  add_footer('Templates', async function() {
    var names = BPI.Template.list();
    if (!names.length) { BPI.Notify.warn('No templates saved yet.'); return; }
    var chosen = await BPI.Notify.prompt(
      'Apply template:',
      names.join(' | ')
    );
    if (chosen && BPI.Template.get(chosen)) {
      BPI.Template.apply(chosen);
      BPI.Notify.success('Template applied: ' + chosen);
    }
  }, { icon: 'ri-file-copy-line', tooltip: 'Apply a saved template' });
});
```

---

### BPI.I18n — Internationalization ★ New

Register per-plugin string dictionaries so your plugin UI is fully translatable. The active language follows the browser locale (`navigator.language`) but can be overridden.

```js
// Register translations for your plugin
BPI.I18n.register('MyPlugin', 'en', {
  greeting: 'Hello, {0}!',
  saved:    'File saved.',
  error:    'Something went wrong: {0}'
});

BPI.I18n.register('MyPlugin', 'es', {
  greeting: '¡Hola, {0}!',
  saved:    'Archivo guardado.',
  error:    'Algo salió mal: {0}'
});

BPI.I18n.register('MyPlugin', 'hi', {
  greeting: 'नमस्ते, {0}!',
  saved:    'फ़ाइल सहेजी गई।',
  error:    'कुछ गलत हुआ: {0}'
});

BPI.I18n.register('MyPlugin', 'fr', {
  greeting: 'Bonjour, {0}!',
  saved:    'Fichier sauvegardé.',
  error:    'Une erreur s\'est produite: {0}'
});

// Translate a key (uses current browser lang, falls back to 'en')
BPI.I18n.t('MyPlugin', 'greeting', ['Alice']);    // "Hello, Alice!" (or localized)
BPI.I18n.t('MyPlugin', 'error',    ['403']);      // "Something went wrong: 403"
BPI.I18n.t('MyPlugin', 'saved');                  // "File saved."

// Override the active language (persisted across sessions)
BPI.I18n.setLang('es');

// Get the current active language code
const lang = BPI.I18n.getLang();   // 'es'

// List all registered language codes across all plugins
const langs = BPI.I18n.available();   // ['en', 'es', 'hi', 'fr']

// List all registered keys for a plugin+language
const keys = BPI.I18n.keys('MyPlugin', 'en');

// Check if a key has a translation
if (BPI.I18n.has('MyPlugin', 'greeting')) { /* … */ }

// Print all translations for a plugin to console
BPI.I18n.dump('MyPlugin');
```

**Example — Localized plugin:**
```js
var P = 'WeatherPlugin';
BPI.I18n.register(P, 'en', { fetch: 'Fetch Weather', done: 'Weather loaded!', fail: 'Request failed.' });
BPI.I18n.register(P, 'es', { fetch: 'Obtener Clima', done: '¡Clima cargado!', fail: 'Solicitud fallida.' });

OnThe_Function('afterBoot', async function() {
  add_footer(BPI.I18n.t(P, 'fetch'), async function() {
    var res = await BPI.Http.get('https://wttr.in/?format=3');
    if (res.ok) {
      BPI.FS.writeFile('weather.txt', res.data);
      BPI.Notify.success(BPI.I18n.t(P, 'done'));
    } else {
      BPI.Notify.error(BPI.I18n.t(P, 'fail'));
    }
  }, { icon: 'ri-cloud-line' });
});
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

### Example 9 — Theme Manager Plugin

```js
// Theme Manager — save, apply and cycle named IDE themes
var THEMES = {
  'Default':   { '--primary':'#3b82f6','--bg-dark':'#09090b','--bg-panel':'#18181b','--bg-editor':'#282c34','--border':'#27272a' },
  'Synthwave': { '--primary':'#e879f9','--bg-dark':'#0d0221','--bg-panel':'#160420','--bg-editor':'#1a0535','--border':'#3b0764' },
  'Forest':    { '--primary':'#4ade80','--bg-dark':'#071207','--bg-panel':'#0c1f0c','--bg-editor':'#102010','--border':'#1a3a1a' },
  'Ocean':     { '--primary':'#22d3ee','--bg-dark':'#030f1a','--bg-panel':'#061624','--bg-editor':'#091c2e','--border':'#0d2a40' },
  'Rose':      { '--primary':'#f43f5e','--bg-dark':'#0f0609','--bg-panel':'#1a0b0f','--bg-editor':'#200d12','--border':'#2e1018' }
};

OnThe_Function('afterBoot', async function() {
  // Save all presets on first load
  Object.entries(THEMES).forEach(function(kv) { BPI.Theme.save(kv[0], kv[1]); });

  add_footer('🎨 Theme', async function() {
    var names = BPI.Theme.list().join(' | ');
    var choice = await BPI.Notify.prompt('Choose theme:', names);
    if (choice && THEMES[choice]) {
      BPI.Theme.apply(choice);
      BPI.Notify.success('Theme: ' + choice);
      BPI.Config.write('ThemeManager', 'active', choice);
    }
  }, { icon: 'ri-palette-line', color: '#a78bfa' });

  // Restore last used theme on boot
  var last = BPI.Config.read('ThemeManager', 'active', null);
  if (last && THEMES[last]) BPI.Theme.apply(last);
});
```

---

### Example 10 — Web Worker Linter

```js
// Run a heavy linting scan in a Web Worker so the UI stays responsive
OnThe_Function('afterBoot', function() {
  add_footer('⚡ Worker Lint', async function() {
    BPI.UI.loadingOverlay(true, 'Scanning in background…');
    try {
      var code = BPI.Editor().getValue();
      var issues = await BPI.Worker.run(function(src) {
        // Runs off the main thread
        var lines = src.split('\n');
        var found = [];
        lines.forEach(function(line, i) {
          if (/\beval\b/.test(line))        found.push({ line: i+1, msg: 'eval() used' });
          if (/\bvar\s/.test(line))         found.push({ line: i+1, msg: 'var (use let/const)' });
          if (/console\.log/.test(line))    found.push({ line: i+1, msg: 'console.log left in' });
          if (/TODO|FIXME/.test(line))      found.push({ line: i+1, msg: 'TODO/FIXME comment' });
          if (line.length > 200)            found.push({ line: i+1, msg: 'Line too long' });
        });
        return found;
      }, code, 8000);
      BPI.UI.loadingOverlay(false);
      if (!issues.length) {
        BPI.Notify.success('✅ No issues found');
      } else {
        BPI.Logger.table('WorkerLint', issues);
        BPI.Notify.warn('⚠️ ' + issues.length + ' issue(s) — see console');
      }
    } catch(e) {
      BPI.UI.loadingOverlay(false);
      BPI.Notify.error('Worker failed: ' + e.message);
    }
  }, { icon: 'ri-flashlight-line', color: '#fbbf24' });
});
```

---

### Example 11 — Multi-Language Plugin (i18n)

```js
// Fully localized plugin that adapts to the user's browser language
var P = 'MultiLang';
BPI.I18n.register(P, 'en', {
  title:   'Code Stats',
  lines:   '{0} lines',
  words:   '{0} words',
  chars:   '{0} chars',
  noFile:  'No file open'
});
BPI.I18n.register(P, 'es', {
  title:   'Estadísticas',
  lines:   '{0} líneas',
  words:   '{0} palabras',
  chars:   '{0} caracteres',
  noFile:  'Sin archivo abierto'
});
BPI.I18n.register(P, 'hi', {
  title:   'कोड आँकड़े',
  lines:   '{0} पंक्तियाँ',
  words:   '{0} शब्द',
  chars:   '{0} अक्षर',
  noFile:  'कोई फ़ाइल नहीं खुली'
});

OnThe_Function('afterBoot', function() {
  function updateChip() {
    var editor = BPI.Editor();
    if (!editor) { BPI.UI.statusChip({ id: 'stats', text: BPI.I18n.t(P, 'noFile'), color: '#555' }); return; }
    var s = BPI.Format.wordCount(editor.getValue());
    BPI.UI.statusChip({
      id:    'stats',
      text:  BPI.I18n.t(P,'lines',[s.lines]) + ' · ' + BPI.I18n.t(P,'words',[s.words]),
      color: '#a78bfa',
      icon:  'ri-bar-chart-line'
    });
  }
  updateChip();
  setInterval(updateChip, 2000);
  BPI.Notify.info(BPI.I18n.t(P,'title') + ' active (lang: ' + BPI.I18n.getLang() + ')');
});
```

---

### Example 12 — Color Accessibility Checker

```js
// Check WCAG contrast on any two colors from the IDE
OnThe_Function('afterBoot', function() {
  add_footer('♿ A11y Check', async function() {
    var fg = await BPI.Notify.prompt('Foreground color (hex):', '#3b82f6');
    if (!fg) return;
    var bg = await BPI.Notify.prompt('Background color (hex):', '#09090b');
    if (!bg) return;

    var ratio   = BPI.Color.contrast(fg, bg);
    var passAA  = BPI.Color.isAccessible(fg, bg, 'AA');
    var passAAA = BPI.Color.isAccessible(fg, bg, 'AAA');
    var readable = BPI.Color.readable(bg);

    BPI.Console().log('=== WCAG Contrast Report ===');
    BPI.Console().log('FG: ' + fg + '   BG: ' + bg);
    BPI.Console().log('Contrast ratio: ' + ratio + ':1');
    BPI.Console().log('AA  (4.5:1) — ' + (passAA  ? '✅ PASS' : '❌ FAIL'));
    BPI.Console().log('AAA (7:1)   — ' + (passAAA ? '✅ PASS' : '❌ FAIL'));
    BPI.Console().log('Readable text on BG: ' + readable);

    var lighterFg = BPI.Color.lighten(fg, 0.2);
    var betterRatio = BPI.Color.contrast(lighterFg, bg);
    if (!passAA) {
      BPI.Console().log('Suggestion: try ' + lighterFg + ' (ratio: ' + betterRatio.toFixed(2) + ':1)');
    }

    BPI.Notify[passAA ? 'success' : 'warn'](
      ratio.toFixed(2) + ':1 — ' + (passAA ? 'AA PASS' : 'AA FAIL')
    );
  }, { icon: 'ri-eye-line', color: '#34d399' });
});
```

---

### Example 13 — Storage-Backed Notes Plugin

```js
// Quick notes plugin that saves to IndexedDB and syncs to a project file
var NOTE_KEY = 'quickNotes:content';

OnThe_Function('afterBoot', async function() {
  // Load existing note from IDB on boot
  var saved = await BPI.Storage.getIDB(NOTE_KEY, '');

  add_footer('📝 Notes', async function() {
    var current = await BPI.Storage.getIDB(NOTE_KEY, '');
    var updated = await BPI.Notify.prompt('Quick Notes (saved automatically):', current || '');
    if (updated === null) return;  // cancelled
    await BPI.Storage.setIDB(NOTE_KEY, updated);
    BPI.FS.writeFile('notes/quick-notes.md', updated);
    BPI.Notify.success('Note saved to IDB + project!');
  }, { icon: 'ri-sticky-note-line', color: '#fbbf24' });

  // Keep a chip showing char count
  BPI.Storage.getIDB(NOTE_KEY).then(function(n) {
    if (n) BPI.UI.statusChip({ id: 'noteCount', text: n.length + 'c note', color: '#fbbf24', icon: 'ri-sticky-note-line' });
  }).catch(function(){});
});
```

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

**9. Use `BPI.Theme.patch()` instead of direct `document.documentElement.style.setProperty()`**
```js
// Bad — bypasses BPI tracking
document.documentElement.style.setProperty('--primary', '#f00');

// Good — logged, reversible, and inspectable via BPI.Theme.current()
BPI.Theme.setVar('--primary', '#f00');
// Or bulk:
BPI.Theme.patch({ '--primary': '#f00', '--bg-editor': '#200' });
```

**10. Use `BPI.Worker.run()` for CPU-heavy operations**
```js
// Bad — freezes the IDE UI
const result = myExpensiveSort(largeArray);

// Good — runs in a background thread, UI stays responsive
const result = await BPI.Worker.run(function(arr) {
  return arr.sort(function(a, b) { return a - b; });
}, largeArray);
```

**11. Use `BPI.Storage.setIDB()` for large or complex data**
```js
// Bad — localStorage has a 5MB limit and blocks the main thread on parse
localStorage.setItem('myPlugin:bigData', JSON.stringify(hugeArray));

// Good — IndexedDB handles large data asynchronously
await BPI.Storage.setIDB('myPlugin:bigData', hugeArray);
```

**12. Always register i18n translations at the top of your plugin, before any UI is built**
```js
// Good — translations registered before add_footer() calls
var P = 'MyPlugin';
BPI.I18n.register(P, 'en', { run: 'Run', cancel: 'Cancel' });
BPI.I18n.register(P, 'es', { run: 'Ejecutar', cancel: 'Cancelar' });

OnThe_Function('afterBoot', function() {
  add_footer(BPI.I18n.t(P, 'run'), runFn);
});
```

**13. Use `BPI.Color.isAccessible()` to ensure your plugin's custom UI colors pass WCAG AA**
```js
var myColor = '#a78bfa';
var bg      = BPI.Theme.getVar('--bg-dark');
if (!BPI.Color.isAccessible(myColor, bg)) {
  myColor = BPI.Color.lighten(myColor, 0.2);  // auto-fix
}
BPI.UI.statusChip({ id: 'myChip', text: 'OK', color: myColor });
```
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

## New in v1.15 — API Additions

Six brand-new BPI modules and numerous extensions to existing ones:

| Module | What's New |
|---|---|
| `BPI.Theme` | Full CSS variable theme control: `setVar`, `getVar`, `save`, `apply`, `patch`, `reset`, `current`, `list`, `isDark` |
| `BPI.Color` | Color math library: `mix`, `lighten`, `darken`, `contrast`, `isAccessible`, `readable`, `toRgba`, `invert`, `randomHex`, `hexToRgb`, `rgbToHex` |
| `BPI.Storage` | Three-layer store: IndexedDB (`setIDB/getIDB/delIDB`), `session.*`, `local.*`, `exportLocal`, `importLocal` |
| `BPI.Worker` | Web Worker pool: `spawn`, `post`, `onMessage`, `terminate`, `terminateAll`, `list`, and one-shot `run(fn, data)` |
| `BPI.Template` | Code template manager: `save`, `apply`, `insert`, `saveFromEditor`, `list`, `rename`, `export`, `import`, `clear` |
| `BPI.I18n` | Plugin localization: `register`, `t` (with `{0}` placeholder substitution), `setLang`, `getLang`, `available`, `keys`, `has`, `dump` |
| `BPI.FS` | Added: `readJson`, `writeJson`, `appendFile`, `moveFile`, `getStats`, `readAll`, `listByExt` |
| `BPI.Execute` | Added: `runFile`, `interval`, `retry`, `debounce`, `throttle` |
| `BPI.Http` | Added: `put`, `patch`, `delete`, `loadCSS`, `getJSON` |
| `BPI.Format` | Added: `toPascalCase`, `toKebabCase`, `htmlEncode`, `htmlDecode`, `truncate`, `stripHtml`, `wrapLines`, `countOccurrences`, `padLines`, `toTable` |
| `BPI.UI` | Added: `tooltip`, `contextMenu`, `badgeCount`, `highlight`, `loadingOverlay` |
| `BPI.Notify` | Added: `progress` (live progress bar), `desktop` (system notification), `queue` (batched toasts) |
| `BPI.Logger` | Added: `table`, `groupStart`, `groupAdd`, `groupEnd` |

---

## Getting Help

| Resource | Link |
|---|---|
| 📧 Email Support | [Help.magmamine@gmail.com](mailto:Help.magmamine@gmail.com) |
| 🌐 Official Website | [beyoneer.xyz](https://beyoneer.xyz) |
| 📟 In-IDE Reference | Run `BPI.help()` in the Beyoneer console |
| 💻 Terminal shortcut | Type `bpi help` in the IDE terminal |

---

*BPI Documentation · Beyoneer IDE v1.15 (Patched) · By **MagmaMinesTeam***  
*© MagmaMinesTeam — [Help.magmamine@gmail.com](mailto:Help.magmamine@gmail.com) — [beyoneer.xyz](https://beyoneer.xyz)*
