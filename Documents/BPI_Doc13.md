# BPI — Beyoneer Programming Interface
### Official Developer Reference · Plugins· Source-Verified & Corrected

> **Beyoneer IDE** is a full-featured, browser-based code editor. The **BPI** (Beyoneer Programming Interface) is its plugin and scripting API — a collection of powerful namespaces that let you extend, automate, and hook into every part of the IDE from JavaScript.
>
> **This document was written directly from the editor source.** Every function name, signature, and behaviour reflects exactly what is defined in `editor` Nothing is invented or inferred.

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
| `onFileSystemChange` | A file is created, deleted, renamed, or moved via BPI.FS | `(action, path)` where action = `'create'` \| `'delete'` \| `'rename'` \| `'move'` |

> ⚠️ **Note:** `onSave`, `onRun`, and `onTabChange` do **not** exist in v1.15. Do not use them.

You can register the same plugin to multiple hooks:

```js
OnThe_Function('afterBoot',           onBoot);
OnThe_Function('onFileOpen',          onFileOpen);
OnThe_Function('beforeSave',          onBeforeSave);
OnThe_Function('onFileSystemChange',  onFSChange);

function onBoot()                          { BPI.Notify.info('Plugin loaded'); }
function onFileOpen(path, content)         { BPI.Console().log('Opened: ' + path); }
function onBeforeSave(projId, proj)        { BPI.Console().log('Saving project: ' + projId); }
function onFSChange(action, path)          { BPI.Console().log(action + ': ' + path); }
```

> **Re-entrancy guard:** `triggerHook` has a built-in guard that prevents any hook from firing itself recursively. If you call `BPI.FS.writeFile` inside `onFileSystemChange`, the nested `onFileSystemChange` trigger is silently suppressed.

---

## The `BPI` Master Namespace

The `BPI` object is the single entry point to all APIs. The lower-level `window.Beyo*_` globals are identical references.

```js
// Direct property aliases:
BPI.FS           === window.BeyoFS
BPI.Execute      === window.BeyoExecute_
BPI.Project      === window.BeyoProject_
BPI.Env          === window.BeyoEnv_
BPI.Clipboard    === window.BeyoClipboard_
BPI.Format       === window.BeyoFormat_
BPI.Search       === window.BeyoSearch_
BPI.Diff         === window.BeyoDiff_
BPI.Lint         === window.BeyoLint_
BPI.Notify       === window.BeyoNotify_
BPI.Snippets     === window.BeyoSnippetLib_
BPI.Http         === window.BeyoHttp_
BPI.UI           === window.BeyoUI_
BPI.Git          === window.BeyoGit_
BPI.Task         === window.BeyoTask_
BPI.Event        === window.BeyoEvent_
BPI.KeyMap       === window.BeyoKeyMap_
BPI.Logger       === window.BeyoLogger_
BPI.Config       === window.BeyoConfig_
BPI.Bench        === window.BeyoBench_
BPI.Theme        === window.BeyoTheme_
BPI.Color        === window.BeyoColor_
BPI.Storage      === window.BeyoStorage_
BPI.Worker       === window.BeyoWorker_
BPI.Template     === window.BeyoTemplate_
BPI.I18n         === window.BeyoI18n_

// Function-call accessors (these are getter functions, not properties):
BPI.Editor()     // → returns window.BeyoEditor_
BPI.Console()    // → returns window.BeyoConsole_
BPI.Debugger()   // → returns window.Debugger
BPI.DIS()        // → returns window.DIS
BPI.DISPLUS()    // → returns window.DISPLUS

// Metadata:
BPI.version      // "1.15"
BPI.help()       // Prints the full formatted reference to the IDE console
```

Call `BPI.help()` in the console at any time for a live formatted reference table.

---

## API Reference

---

### BPI.FS — File System

Read, write, and manage files in the current project. Backed by `window.BeyoFS`.

> ⚠️ **Corrected method names** — the table at the end of this section lists the wrong names that appear in older docs.

```js
// Read a file — throws if the file is not found
const content = BPI.FS.readFile('src/index.js');

// Write (create or overwrite) a file
BPI.FS.writeFile('output/result.txt', 'Hello, world!');

// Delete a file
BPI.FS.deleteFile('temp/scratch.js');

// List all files — optional filter is a substring or extension match
const allFiles  = BPI.FS.listFiles();          // all files
const jsFiles   = BPI.FS.listFiles('.js');     // only files ending in .js
const srcFiles  = BPI.FS.listFiles('src/');    // only files containing 'src/'

// Check if a file exists (returns boolean)
if (BPI.FS.fileExists('config.json')) {
  const cfg = JSON.parse(BPI.FS.readFile('config.json'));
}

// Get file metadata — returns { path, size, type, modified } or null
const info = BPI.FS.getFileInfo('src/app.js');
BPI.Console().log('Size: ' + info.size + ' bytes');

// Copy a file
BPI.FS.copyFile('src/template.js', 'src/myComponent.js');

// Rename a file (in-place) — fires onFileSystemChange('rename', newPath)
BPI.FS.renameFile('old-name.js', 'new-name.js');

// Get the current project name as a string
const name = BPI.FS.getProjectName();
```

**Extended methods (added v1.15):**

```js
// Read a JSON file and parse it — 2nd arg is fallback if file missing/invalid
const cfg = BPI.FS.readJson('config.json', {});

// Write any JSON-serializable value to a file (2-space indent by default)
BPI.FS.writeJson('data/result.json', { ts: Date.now(), ok: true });

// Append text to a file (creates the file if it does not exist)
BPI.FS.appendFile('logs/debug.log', new Date().toISOString() + ' — event\n');

// Move a file to a new path — fires onFileSystemChange('move', dest)
BPI.FS.moveFile('src/old-name.js', 'src/new-name.js');

// Get word / line / char stats — returns { path, lines, words, chars, type }
const stats = BPI.FS.getStats('src/app.js');

// Read ALL project files into a { path: content } map
const all   = BPI.FS.readAll();        // every file
const jsAll = BPI.FS.readAll('.js');   // only .js files

// Count lines in a file (returns 0 if file not found)
const n = BPI.FS.lineCount('src/app.js');

// List files filtered by extension array — pass extensions without leading dot
const webFiles = BPI.FS.listByExt(['html', 'css', 'js']);
```

**Corrected method name table:**

| ❌ Wrong (old docs) | ✅ Correct |
|---|---|
| `BPI.FS.list()` | `BPI.FS.listFiles()` |
| `BPI.FS.exists(path)` | `BPI.FS.fileExists(path)` |
| `BPI.FS.copy(a, b)` | `BPI.FS.copyFile(a, b)` |
| `BPI.FS.rename(a, b)` | `BPI.FS.renameFile(a, b)` |
| `BPI.FS.delete(p)` | `BPI.FS.deleteFile(p)` |
| `BPI.FS.size(p)` | `BPI.FS.getFileInfo(p).size` |

---

### BPI.Editor() — Editor Control

Direct access to the CodeMirror 6 editor instance. Note: this is a **function call**, not a property — it returns `window.BeyoEditor_`.

```js
const editor = BPI.Editor();

// Get all text in the editor
const code = editor.getValue();

// Replace all editor content (also persists to the project)
editor.setValue('// New content\nconsole.log("hi");');

// Insert text at the cursor position (mobile-safe)
editor.insertAtCursor('// inserted here\n');

// Wrap the current selection with a before/after string
editor.wrapSelection('/**', '*/');

// Get the currently selected text
const selection = editor.getSelectedText();

// Get current cursor position — returns character OFFSET (a number), not {line,col}
const offset = editor.getCursor();

// Get the current line — returns { text: string, number: int }
const line = editor.getLine();   // e.g. { text: '  return x;', number: 42 }

// Get the currently open file path
const filename = editor.getActiveFile();   // e.g. 'src/app.js'

// Get the file extension of the active file (lowercase, no dot)
const ext = editor.getActiveExt();         // e.g. 'js'

// Open a file by path (triggers onFileOpen hook)
editor.openFile('src/utils.js');

// Save the active file to the project explicitly
editor.saveActiveFile();

// Programmatically create a new project file
editor.createFile('src/newFile.js', '// content here');

// Prepend a line to the top of the editor content
editor.prependLine('// AUTO-GENERATED');

// Append a line to the bottom of the editor content
editor.appendLine('// end of file');

// Replace all occurrences of a string in the editor
editor.replaceAll('oldFunctionName', 'newFunctionName');

// Convenience proxies to BPI.FS (same behaviour)
editor.listFiles('.js');
editor.readFile('src/app.js');
editor.writeFile('src/app.js', code);

// Trigger Prettier formatting on the current file (async)
editor.formatCode();
```

> ⚠️ There is no `gotoLine()` method on `BeyoEditor_`. To jump to a line, use `BeyoMobile_.selectLines(n, n)` or compute the character offset from `editorView.state.doc.line(n).from` manually.

> ⚠️ Always guard before use: `const editor = BPI.Editor(); if (!editor) return;`

---

### BPI.Console() — Console Output

Write to the IDE's built-in console panel. This is a **function call** — it returns `window.BeyoConsole_`.

```js
const con = BPI.Console();

con.log('Normal message');                   // info style
con.warn('Something might be wrong');        // warning style
con.error('Something went wrong!');          // ✅ .error() works — patched in v1.15
con.err('Also works');                       // legacy alias (kept for compatibility)
con.dim('Quiet / secondary output');         // dimmed style
con.clear();                                 // ✅ clears the console panel — patched in v1.15
```

> ⚠️ In the original unpatched v1.15 release, `.error()` was silently broken — it did not exist. Only `.err()` worked. The patched build adds `.error()` as a proper alias. If you are on an unpatched IDE, use `.err()` as a fallback.

---

### BPI.Execute — Code Execution

Run JavaScript code strings, schedule tasks, and benchmark performance. Backed by `window.BeyoExecute_`.

```js
// Run a JavaScript string — returns { ok, result, ms } or { ok: false, error }
const res = BPI.Execute.run('1 + 2 + 3');
// Logs: [BeyoExec] ✓ Done in 0.12ms → 6

// Run with a custom label (shows in the console prefix)
BPI.Execute.run('console.log("hi")', { label: 'MyPlugin' });

// Run the entire current editor file
BPI.Execute.runCurrentFile();

// Run only the selected text in the editor
BPI.Execute.runSelection();

// Schedule a function after a delay (ms) — returns the setTimeout id
const id = BPI.Execute.schedule(function() {
  BPI.Notify.info('5 seconds passed!');
}, 5000);

// Benchmark a sync function N times — returns { avg, min, max } (ms)
const bench = BPI.Execute.benchmark(function() {
  let x = 0; for (let i = 0; i < 1000; i++) x += i;
}, 100);

// Run an async function with a timeout guard (default 5000ms)
// Returns { ok: true, result } or { ok: false, error }
const result = await BPI.Execute.runAsync(async function() {
  const r = await fetch('https://api.example.com/data');
  return r.json();
}, 8000);
if (result.ok) BPI.Console().log(JSON.stringify(result.result));
```

**Extended methods (added v1.15):**

```js
// Execute a project file by path — reads it via BPI.FS then runs it
BPI.Execute.runFile('scripts/build.js');

// Repeating interval — returns a stop() function to cancel it
const stop = BPI.Execute.interval(function() {
  BPI.UI.statusChip({ id: 'clock', text: new Date().toLocaleTimeString(), color: '#a78bfa' });
}, 1000, 'clock-label');
// Call stop() to cancel:
stop();

// Retry an async function up to N times with a delay between attempts
// Returns { ok, result, attempts } or { ok: false, attempts }
const r = await BPI.Execute.retry(async () => {
  const res = await BPI.Http.get('https://api.example.com/data');
  if (!res.ok) throw new Error('HTTP ' + res.status);
  return res.data;
}, 3, 500, 'fetch-retry');

// Debounce — returns a wrapper that only fires after a pause in calls
const debouncedLint = BPI.Execute.debounce(() => BPI.Lint.lintCurrent(), 500);

// Throttle — returns a wrapper that fires at most once per interval
const throttledSave = BPI.Execute.throttle(() => BPI.Git.commit('auto'), 2000);
```

---

### BPI.Project — Project Management

Backed by `window.BeyoProject_`.

```js
// Get info about the currently open project
const info = BPI.Project.info();
// { id: 'proj_123', name: 'My App', fileCount: 5, files: ['index.js', ...] }

// List all projects in the IDE
const all = BPI.Project.list();
// [{ id, name, fileCount }, ...]

// Switch to another project by ID or name
BPI.Project.switchTo('My Other Project');

// Create a new project with an optional file template map
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

Persistent key-value store backed by `localStorage`. Data survives page reloads. Backed by `window.BeyoEnv_`.

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

// Clear all env vars (destructive)
BPI.Env.clear();
```

---

### BPI.Clipboard — Clipboard

Backed by `window.BeyoClipboard_`.

```js
// Copy any text to the clipboard
await BPI.Clipboard.copy('Hello, clipboard!');

// Copy the full editor content
await BPI.Clipboard.copyEditor();

// Copy only the selected text in the editor
await BPI.Clipboard.copySelection();

// Paste clipboard text into the editor at the cursor position
await BPI.Clipboard.pasteToEditor();

// Read text from the clipboard (returns string or null on permission error)
const text = await BPI.Clipboard.read();

// Copy a project file's content by path
await BPI.Clipboard.copyFile('src/utils.js');
```

---

### BPI.Format — Code Formatting & Transforms

Backed by `window.BeyoFormat_`.

```js
// Format the current file using Prettier (async — requires Prettier to be loaded)
await BPI.Format.formatCurrent();

// Minify a JS string (strips comments and collapses whitespace)
const minified = BPI.Format.minify(code);

// Convert tabs ↔ spaces (n = spaces per tab, default 2)
const spaced = BPI.Format.tabsToSpaces(code, 2);
const tabbed = BPI.Format.spacesToTabs(code, 2);

// Case conversion
BPI.Format.camelToSnake('myVariableName');   // → "my_variable_name"
BPI.Format.snakeToCamel('my_variable_name'); // → "myVariableName"

// Base64 encode / decode
const encoded = BPI.Format.toBase64('Hello!');
const decoded = BPI.Format.fromBase64(encoded);

// Sort lines alphabetically
const sorted  = BPI.Format.sortLines(code);

// Remove duplicate lines
const deduped = BPI.Format.dedupLines(code);

// Count words, lines, and characters — returns { words, lines, chars }
const stats = BPI.Format.wordCount(code);
```

**Extended methods (added v1.15):**

```js
// Convert to PascalCase
BPI.Format.toPascalCase('my component name');   // → "MyComponentName"

// Convert to kebab-case
BPI.Format.toKebabCase('myVariableName');        // → "my-variable-name"

// HTML entity encode / decode
const safe = BPI.Format.htmlEncode('<script>alert("xss")</script>');
const back = BPI.Format.htmlDecode(safe);

// Truncate a string with ellipsis (max chars, default 80)
BPI.Format.truncate('A very long description text...', 40);

// Strip all HTML tags from a string
BPI.Format.stripHtml('<p>Hello <b>world</b></p>');   // → "Hello world"

// Wrap each line with a prefix and/or suffix
BPI.Format.wrapLines(code, '// ', '');   // comment out every line

// Count occurrences of a substring
BPI.Format.countOccurrences(code, 'console.log');

// Pad all lines to the same width
BPI.Format.padLines(code, ' ');

// Format an array of objects as a console-table string
const table = BPI.Format.toTable([
  { file: 'app.js',   lines: 120 },
  { file: 'main.css', lines:  88 }
]);
BPI.Console().log(table);
```

---

### BPI.Search — Find & Replace

Backed by `window.BeyoSearch_`.

```js
// Find all occurrences in the current file
// Returns [{ index, line, col, match }, ...]
const matches = BPI.Search.findAll('console.log');

// Case-sensitive search (second argument)
const exact = BPI.Search.findAll('myVar', true);

// Count occurrences in the current file
const count = BPI.Search.count('TODO');

// Replace all in the editor — returns the number of replacements made
const replaced = BPI.Search.replaceAll('var ', 'const ');

// Search across all project files
// Returns [{ path, line, text }, ...]
const results = BPI.Search.findInProject('fetchData');
```

---

### BPI.Diff — File Diff

Line-level diff utility. Backed by `window.BeyoDiff_`.

```js
// Diff two strings — logs to console, returns { added, removed, changed, lines }
const result = BPI.Diff.diff(oldCode, newCode, 'Before', 'After');

// Diff two project files by path
BPI.Diff.diffFiles('src/app-old.js', 'src/app-new.js');

// Diff the editor's current (unsaved) content vs the saved file
BPI.Diff.diffEditorVsFile('src/app.js');
```

---

### BPI.Lint — Code Linting

JS/TS pattern-based linter. Backed by `window.BeyoLint_`.

```js
// Check JS syntax by attempting a Function parse — { ok: true } or { ok: false, error }
const check = BPI.Lint.checkSyntax('function hello( { return 1; }');

// Lint the current file — reports to console, shows a toast
// Returns [{ line, msg }, ...]
const issues = BPI.Lint.lintCurrent();

// Lint all JS/TS files in the project
BPI.Lint.lintProject();
```

**What `BeyoLint_` detects:**

- Lines longer than 200 characters
- `eval()` usage
- `console.log` left in code
- `debugger` statements
- `TODO` / `FIXME` / `HACK` comments
- `var` declarations (suggests `let`/`const`)

---

### BPI.Notify — Notifications

Rich notification system. Backed by `window.BeyoNotify_`.

> ⚠️ **Bug fix:** In the original IDE, the `duration` parameter passed to `Toast_notification_` was silently ignored. This is fixed in the v1.15 patch. All methods below respect `duration` correctly.

```js
// Toast notifications (show for duration ms then auto-dismiss)
BPI.Notify.success('File saved!');             // default 3000ms, prefix ✅
BPI.Notify.error('Something went wrong');      // default 4000ms, prefix ❌
BPI.Notify.warn('Are you sure?');              // default 3000ms, prefix ⚠️
BPI.Notify.info('Tip: press Ctrl+S to save');  // default 2500ms, prefix ℹ️

// Custom duration (ms)
BPI.Notify.success('Long message!', 6000);

// Vibrate (mobile only, pattern in ms)
BPI.Notify.vibrate([50, 30, 50]);

// Confirm dialog — async, returns true (Confirm) or false (Cancel)
const confirmed = await BPI.Notify.confirm(
  'Delete File',
  'This will permanently delete <strong>index.js</strong>.'
);
if (confirmed) BPI.FS.deleteFile('index.js');

// Input prompt dialog — async, returns string or null if cancelled
const name = await BPI.Notify.prompt('New project name:', 'My App');
if (name) BPI.Project.create(name);
```

**Extended methods (added v1.15):**

```js
// Live progress bar toast — call repeatedly, auto-dismisses ~1.2s after 100%
BPI.Notify.progress(0,   'Building…');
BPI.Notify.progress(50,  'Building…');
BPI.Notify.progress(100, 'Building…');

// Request browser notification permission and send a system notification
// Returns true if the notification was sent, false if permission denied
const sent = await BPI.Notify.desktop('Build Complete', 'All files compiled successfully.');

// Queue multiple toasts with a delay between each
BPI.Notify.queue([
  { text: 'Step 1: Linting…',    type: 'info'    },
  { text: 'Step 2: Formatting…', type: 'info'    },
  { text: '✅ All done!',         type: 'success' }
], 900);   // 900ms between each
// Strings are also accepted: BPI.Notify.queue(['msg1', 'msg2'], 500)
```

---

### BPI.Snippets — Snippet Library

Save and reuse named code snippets. Has 10 built-in starter snippets seeded on first boot. Backed by `window.BeyoSnippetLib_`.

```js
// Save a snippet
BPI.Snippets.save('fetchHelper', `
async function fetchJSON(url) {
  const res = await fetch(url);
  return res.json();
}`, ['utils', 'async'], { lang: 'js', desc: 'Async fetch utility', prefix: 'fetchj' });

// Get the full snippet object
const snip = BPI.Snippets.get('fetchHelper');

// Get only the code string
const code = BPI.Snippets.getCode('fetchHelper');

// Insert a snippet at the cursor position
BPI.Snippets.insert('fetchHelper');

// Save the current editor content (or selection) as a snippet
BPI.Snippets.saveFromEditor('myBoilerplate', { lang: 'js', prefix: 'myb' });

// List all saved snippets — optionally filter by lang or tag
const all  = BPI.Snippets.list();
const jsSnips = BPI.Snippets.list({ lang: 'js' });
const asyncSnips = BPI.Snippets.list({ tag: 'async' });
// Returns array of { name, code, tags, lang, desc, prefix, saved, builtin? }

// Search snippets by name, tag, description, or code content
const results = BPI.Snippets.search('async');

// Copy snippet code to clipboard
await BPI.Snippets.copy('fetchHelper');

// Rename a snippet
BPI.Snippets.rename('fetchHelper', 'jsonFetcher');

// Delete a snippet
BPI.Snippets.delete('jsonFetcher');

// Dump all snippet names to the IDE console
BPI.Snippets.dump();

// Export all snippets as a JSON file download
BPI.Snippets.export();

// Import snippets from a JSON string (merges with existing)
BPI.Snippets.import(jsonStr);

// Open the visual Snippet Manager dialog
BPI.Snippets.showManager();
```

> **Prefix trigger:** If a snippet has a `prefix` set, typing that prefix in the editor and pressing **Tab** inserts the snippet automatically.

---

### BPI.Http — HTTP Requests

Fetch wrapper for external APIs. Backed by `window.BeyoHttp_`.

All methods return `{ ok: boolean, status: number, data: any }` (or `{ ok: false, error: string }` on network failure). The `data` field is auto-parsed as JSON when the response body is valid JSON; otherwise it is a raw string.

```js
// GET request
const res = await BPI.Http.get('https://api.github.com/users/octocat');
if (res.ok) BPI.Console().log('Name: ' + res.data.name);

// GET with custom headers
const auth = await BPI.Http.get(
  'https://api.example.com/profile',
  { 'Authorization': 'Bearer ' + BPI.Env.get('TOKEN') }
);

// POST request (body auto-serialized to JSON, Content-Type set automatically)
const post = await BPI.Http.post('https://api.example.com/items', {
  title: 'New Item',
  done: false
});

// Dynamically load an external script (cached — won't double-load same URL)
await BPI.Http.loadScript('https://cdn.example.com/chart.min.js');
```

**Extended methods (added v1.15):**

```js
// PUT request
const put = await BPI.Http.put('https://api.example.com/items/1', { done: true });

// PATCH request
const patch = await BPI.Http.patch('https://api.example.com/items/1', { title: 'Updated' });

// DELETE request — returns { ok, status } only
const del = await BPI.Http.delete('https://api.example.com/items/1');

// Load a CSS stylesheet dynamically (optional id prevents double-loading)
await BPI.Http.loadCSS(
  'https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css',
  'animate'
);

// Shorthand JSON GET — returns the parsed object, or null on failure
const user = await BPI.Http.getJSON('https://api.github.com/users/octocat');
if (user) BPI.Console().log('Login: ' + user.login);
```

---

### BPI.UI — Dynamic UI

Build panels, chips, tooltips, and overlays from plugin code. Backed by `window.BeyoUI_`.

```js
// Create a draggable floating panel
BPI.UI.floatingPanel({
  id:    'myPanel',
  title: 'My Plugin Panel',
  html:  '<p>Hello!</p><button onclick="BPI.Notify.success(\'Hi!\')">Click</button>',
  x:     60,     // initial left offset (px)
  y:     100,    // initial top offset (px)
  width: 340     // panel width (px)
});

// Close (remove) a floating panel by id
BPI.UI.closePanel('myPanel');

// Add or update a status chip in the IDE status bar
BPI.UI.statusChip({
  id:    'myStatus',
  text:  '● Connected',
  color: '#4ade80',
  icon:  'ri-wifi-line'    // RemixIcon class
});

// Remove a status chip
BPI.UI.removeChip('myStatus');
```

**Extended methods (added v1.15):**

```js
// Show a tooltip anchored below any element (auto-removes after duration ms)
BPI.UI.tooltip('myButtonId', 'Click to format your file', 2500);

// Build and show a right-click context menu at an (x, y) coordinate
BPI.UI.contextMenu({
  x: 200, y: 300,
  items: [
    { label: 'Format File', icon: 'ri-magic-line', action: () => BPI.Format.formatCurrent() },
    { label: 'Lint File',   icon: 'ri-bug-line',   action: () => BPI.Lint.lintCurrent()     },
    'sep',                  // inserts a visual separator line
    { label: 'Word Count',  icon: 'ri-text',        action: () => {
      const s = BPI.Format.wordCount(BPI.Editor().getValue());
      BPI.Notify.info(s.lines + ' lines, ' + s.words + ' words');
    }}
  ]
});

// Show / update a numeric badge on an existing status chip (0 removes the badge)
BPI.UI.statusChip({ id: 'errors', text: 'Errors', color: '#f87171', icon: 'ri-error-warning-line' });
BPI.UI.badgeCount('errors', 5);   // shows "5"; set to 0 to remove

// Flash a brief glow highlight on any element by DOM id
BPI.UI.highlight('myButtonId', '#3b82f6', 800);   // color, duration ms

// Show a full-screen loading overlay with a message
BPI.UI.loadingOverlay(true, 'Compiling…');
// ... async work ...
BPI.UI.loadingOverlay(false);
```

---

### BPI.Git — Version Control

Lightweight in-session Git: snapshot, compare, and restore editor content. History is capped at 50 commits per file and persisted in `localStorage`. Backed by `window.BeyoGit_`.

```js
// Commit (snapshot) the current editor content with a message
BPI.Git.commit('Before big refactor');

// List all commits for the current file (also prints to console)
// Returns [{ message, code, ts }, ...]
const log = BPI.Git.log();

// List commits for a specific file by path
const log2 = BPI.Git.log('src/utils.js');

// Restore a snapshot by index (0-based)
BPI.Git.checkout(0);    // first commit
BPI.Git.checkout();     // latest commit (no argument = last)

// Diff the current editor content vs the most recent commit
BPI.Git.diffHead();

// Clear all commit history for the current file
BPI.Git.reset();

// Clear history for a specific file
BPI.Git.reset('src/utils.js');

// List all files that have any commit history
const tracked = BPI.Git.status();
```

---

### BPI.Task — Background Tasks

Named async task queue. Backed by `window.BeyoTask_`.

```js
// Register a named async task function
BPI.Task.register('buildProject', async function() {
  BPI.UI.loadingOverlay(true, 'Building…');
  await BPI.Execute.runAsync(async () => {
    // ... do build work ...
  }, 30000);
  BPI.UI.loadingOverlay(false);
});

// Run a registered task (returns the task's promise)
await BPI.Task.run('buildProject');

// Check if a task is currently running
if (BPI.Task.isRunning('buildProject')) {
  BPI.Notify.warn('Build already in progress!');
}

// List all registered task names (also prints to console)
const names = BPI.Task.list();

// Unregister a task
BPI.Task.unregister('buildProject');
```

---

### BPI.Event — Event Bus

IDE-wide pub/sub between plugins. Backed by `window.BeyoEvent_`.

```js
// Subscribe to a named event — returns a handler id for later unsubscribe
const subId = BPI.Event.on('myPlugin:ready', function(data) {
  BPI.Notify.success('Got ' + data.count + ' items');
});

// Subscribe once (auto-unsubscribes after the first fire)
BPI.Event.once('myPlugin:ready', function(data) {
  BPI.Console().log('First fire: ' + JSON.stringify(data));
});

// Emit an event with a payload
BPI.Event.emit('myPlugin:ready', { count: 42, items: ['a', 'b', 'c'] });

// Unsubscribe by the id returned from .on()
BPI.Event.off('myPlugin:ready', subId);

// List all event channel names that have active listeners
BPI.Event.channels();

// Remove all listeners for a channel
BPI.Event.clear('myPlugin:ready');
```

---

### BPI.KeyMap — Keyboard Shortcuts

Global keyboard shortcut registry. Backed by `window.BeyoKeyMap_`.

Combo strings are normalized to uppercase and spaces stripped — `'Ctrl+Shift+F'` and `'ctrl+shift+f'` are equivalent.

```js
// Bind a shortcut (combo, handler function, description)
BPI.KeyMap.bind('Ctrl+Shift+F', function() {
  BPI.Format.formatCurrent();
}, 'Format current file');

BPI.KeyMap.bind('Ctrl+Shift+L', function() {
  BPI.Lint.lintCurrent();
}, 'Lint current file');

// Unbind a shortcut
BPI.KeyMap.unbind('Ctrl+Shift+F');

// List all registered shortcuts (also prints to console)
// Returns [{ combo, description }, ...]
const bindings = BPI.KeyMap.list();
```

---

### BPI.Logger — Structured Logging

Tagged, levelled logging with a queryable in-memory history (max 1000 entries). Backed by `window.BeyoLogger_`.

```js
// Log at different levels — format: [HH:mm:ss.mmm] [LEVEL] [tag] message
BPI.Logger.info('MyPlugin',  'Initialization complete');
BPI.Logger.warn('MyPlugin',  'Config missing — using defaults');
BPI.Logger.error('MyPlugin', 'API call failed: 503');
BPI.Logger.debug('MyPlugin', 'Processing item #42');

// Query log history — returns array of { level, tag, msg, ts }
const errors = BPI.Logger.query({ level: 'ERROR', limit: 20 });
const myLogs  = BPI.Logger.query({ tag: 'MyPlugin', limit: 50 });

// Print summary to console
BPI.Logger.summary();
// Example: [BeyoLogger] Summary — INFO:5 WARN:2 ERROR:1 DEBUG:8 | Total:16

// Export all logs as a JSON string
const json = BPI.Logger.export();

// Clear all log history
BPI.Logger.clear();
```

**Extended methods (added v1.15):**

```js
// Log an array of objects as a formatted table to the IDE console
BPI.Logger.table('FileAudit', [
  { file: 'index.js', lines: 120, size: '4.2kb' },
  { file: 'app.css',  lines:  88, size: '2.1kb' }
]);

// Group related log entries under a named label
BPI.Logger.groupStart('Build', 'transpile');
BPI.Logger.groupAdd('Build',   'transpile', 'Compiling TypeScript…');
BPI.Logger.groupAdd('Build',   'transpile', 'Bundling modules…');
const entries = BPI.Logger.groupEnd('Build', 'transpile');
// entries = ['Compiling TypeScript…', 'Bundling modules…']
```

---

### BPI.Config — Plugin Config Storage

Persistent per-plugin config in `localStorage`, automatically namespaced under `bpi_cfg_<pluginId>`. Backed by `window.BeyoConfig_`.

```js
// Save config (merges into existing — does not overwrite other keys)
BPI.Config.set('MyPlugin', { theme: 'dark', fontSize: 14, autoSave: true });

// Read the full config object (returns {} if nothing saved)
const cfg = BPI.Config.get('MyPlugin');

// Read a single key with fallback
const theme = BPI.Config.read('MyPlugin', 'theme', 'light');

// Write a single key
BPI.Config.write('MyPlugin', 'fontSize', 16);

// Reset (delete) all config for a plugin
BPI.Config.reset('MyPlugin');

// List all plugin IDs that have saved config (also prints to console)
const ids = BPI.Config.list();
```

---

### BPI.Bench — Performance Benchmarking

Backed by `window.BeyoBench_`.

```js
// Named timer
BPI.Bench.start('myOperation');
// ... do work ...
const ms = BPI.Bench.end('myOperation');
// Logs: [BeyoBench] ⏱ myOperation: 12.345ms

// Benchmark a sync function N times — returns avg ms per iteration
const avgMs = BPI.Bench.run('sort', function() {
  [5, 3, 1, 4, 2].sort(function(a, b) { return a - b; });
}, 1000);

// Benchmark an async function N times — returns avg ms per iteration
const avgAsync = await BPI.Bench.runAsync('fetchPing', async function() {
  await fetch('/api/ping');
}, 10);

// Report JS heap memory (Chrome only) — returns { usedMB, limitMB } or null
const mem = BPI.Bench.memory();
```

---

### BPI.Theme — IDE Theme Control ★ New

Read, modify, save and restore the IDE's CSS variable theme from plugin code. Backed by `window.BeyoTheme_`.

**Default CSS variables tracked by BPI.Theme:**

| Variable | Default |
|---|---|
| `--bg-dark` | `#09090b` |
| `--bg-panel` | `#18181b` |
| `--bg-editor` | `#282c34` |
| `--tab-bg` | `#1f1f22` |
| `--tab-active` | `#282c34` |
| `--primary` | `#3b82f6` |
| `--text-main` | `#f4f4f5` |
| `--text-muted` | `#a1a1aa` |
| `--border` | `#27272a` |
| `--folder-color` | `#fbbf24` |
| `--error` | `#f87171` |
| `--success` | `#4ade80` |
| `--warn` | `#fbbf24` |

```js
// Set a single CSS variable instantly
BPI.Theme.setVar('--primary', '#e879f9');
BPI.Theme.setVar('--bg-editor', '#1a0535');

// Get a variable's current computed value
const accent = BPI.Theme.getVar('--primary');   // e.g. '#3b82f6'

// Snapshot the current theme under a name (saves to localStorage)
BPI.Theme.save('myTheme');

// Apply a saved theme (restores all its variables)
BPI.Theme.apply('myTheme');

// List saved theme names (also prints to console)
const themes = BPI.Theme.list();   // ['myTheme', 'synthwave', ...]

// Delete a saved theme
BPI.Theme.delete('myTheme');

// Bulk-apply multiple variables without saving
BPI.Theme.patch({
  '--primary':   '#e879f9',
  '--bg-dark':   '#0d0221',
  '--bg-panel':  '#160420',
  '--bg-editor': '#1a0535',
  '--border':    '#3b0764'
});

// Get all currently computed variable values as a plain object
const current = BPI.Theme.current();

// Reset all variables to Beyoneer defaults
BPI.Theme.reset();

// Check if the IDE is in dark mode (true when --bg-dark luminance < #888888)
const dark = BPI.Theme.isDark();

// Get the default variable map (for reference)
const defaults = BPI.Theme.defaults();
```

> `BPI.Theme` is the full programmatic API. The `BeyoTheme_({…})` global (see Extended Globals) is a legacy shim that only accepts `{ primary, bg, panel, editor, border, success, error, font }` — both work simultaneously.

---

### BPI.Color — Color Utilities ★ New

A full color math library for generating palettes, checking WCAG accessibility, and converting formats. Backed by `window.BeyoColor_`.

```js
// Parse a hex string → { r, g, b }
const rgb = BPI.Color.hexToRgb('#3b82f6');    // { r: 59, g: 130, b: 246 }

// { r, g, b } → hex string
const hex = BPI.Color.rgbToHex(59, 130, 246); // '#3b82f6'

// Blend two hex colors by a 0–1 ratio (0 = full A, 1 = full B)
const mid = BPI.Color.mix('#3b82f6', '#e879f9', 0.5);

// Lighten / darken by a 0–1 amount (blends toward #fff or #000)
const lighter = BPI.Color.lighten('#3b82f6', 0.3);
const darker  = BPI.Color.darken('#3b82f6', 0.3);

// WCAG relative luminance (returns 0–1 float)
const lum = BPI.Color.luminance('#3b82f6');

// WCAG contrast ratio between two hex colors
const ratio = BPI.Color.contrast('#3b82f6', '#09090b');   // e.g. 5.74

// Check WCAG AA (4.5:1) or AAA (7:1) accessibility
const okAA  = BPI.Color.isAccessible('#3b82f6', '#09090b');         // default AA
const okAAA = BPI.Color.isAccessible('#ffffff', '#000000', 'AAA');

// Get the best readable text color (black or white) for a given background
const textColor = BPI.Color.readable('#3b82f6');   // '#ffffff' or '#000000'

// Convert hex to a CSS rgba() string
const rgba = BPI.Color.toRgba('#3b82f6', 0.5);   // 'rgba(59,130,246,0.5)'

// Invert a hex color
const inv = BPI.Color.invert('#3b82f6');   // '#c47d09'

// Generate a random hex color
const rand = BPI.Color.randomHex();   // e.g. '#a3f291'
```

---

### BPI.Storage — Enhanced Storage ★ New

Three-layer persistent storage: **IndexedDB** (async, large data), **sessionStorage** (tab-scoped), and **localStorage** (persistent). Backed by `window.BeyoStorage_`.

```js
// ── IndexedDB (async — survives reloads, best for large data) ──────────

await BPI.Storage.setIDB('myPlugin:cache', { items: [1,2,3], ts: Date.now() });
// Returns true on success, false on failure

const data = await BPI.Storage.getIDB('myPlugin:cache');
// Returns stored value, or null (fallback) if key does not exist

await BPI.Storage.delIDB('myPlugin:cache');

const keys = await BPI.Storage.keysIDB();   // all IDB keys

await BPI.Storage.clearIDB();               // ⚠️ wipes the entire IDB store

// ── Session Storage (cleared when the tab closes) ─────────────────────

BPI.Storage.session.set('draft', { code: '…', ts: Date.now() });
const draft = BPI.Storage.session.get('draft', null);  // 2nd arg = fallback
BPI.Storage.session.del('draft');
const sessionKeys = BPI.Storage.session.keys();
const hasDraft    = BPI.Storage.session.has('draft');
BPI.Storage.session.clear();   // wipe all session keys

// ── Local Storage (persists across reloads) ───────────────────────────

BPI.Storage.local.set('myPlugin:prefs', { theme: 'dark', fontSize: 14 });
const prefs = BPI.Storage.local.get('myPlugin:prefs', {});
BPI.Storage.local.del('myPlugin:prefs');
BPI.Storage.local.clear('myPlugin:');   // clear only keys starting with 'myPlugin:'
const localKeys = BPI.Storage.local.keys();
const hasPrefs  = BPI.Storage.local.has('myPlugin:prefs');

// ── Bulk export / import ──────────────────────────────────────────────

// Export all localStorage entries (or a prefix-filtered subset) to JSON string
const json = BPI.Storage.exportLocal('bpi');           // only 'bpi*' keys
BPI.FS.writeFile('backup/storage.json', json);

// Import a previously-exported JSON string back into localStorage
// Returns true on success, false on failure
BPI.Storage.importLocal(BPI.FS.readFile('backup/storage.json'));
```

---

### BPI.Worker — Web Worker Pool ★ New

Spawn, communicate with, and terminate Web Workers to keep expensive work off the main thread. Backed by `window.BeyoWorker_`.

```js
// ── One-shot: run a function in a worker, get the result ──────────────
const sum = await BPI.Worker.run(function(data) {
  // This code executes in a separate thread — no DOM, no BPI
  let s = 0;
  for (let i = 0; i < data.n; i++) s += i;
  return s;
}, { n: 1000000 }, 10000);   // optional timeout ms (default 10000)
BPI.Console().log('Sum: ' + sum);

// ── Persistent worker: spawn → post → listen → terminate ─────────────

// Spawn from an inline function (wraps it in self.onmessage)
BPI.Worker.spawn('parser', function(data) {
  return data.split('\n').length;
});

// Listen for messages back from the worker
BPI.Worker.onMessage('parser', function(lineCount) {
  BPI.Notify.info('Lines: ' + lineCount);
});

// Post data to the worker
BPI.Worker.post('parser', BPI.Editor().getValue());

// Terminate a specific worker (also revokes the Blob URL)
BPI.Worker.terminate('parser');

// Terminate all active workers at once
BPI.Worker.terminateAll();

// List active worker IDs (also prints to console)
const ids = BPI.Worker.list();
```

> Workers cannot access `window`, `document`, or BPI. They can use `self`, `fetch`, `setTimeout`, `crypto`, and other standard Web Worker APIs.

---

### BPI.Template — Code Template Manager ★ New

Save, retrieve, and apply reusable code templates. Persisted in `localStorage`. Backed by `window.BeyoTemplate_`.

```js
// Save a template (name, code, optional metadata)
BPI.Template.save('fetchHelper', `
async function fetchJSON(url) {
  const res = await fetch(url);
  if (!res.ok) throw new Error(res.status);
  return res.json();
}`, { lang: 'js', tags: ['async', 'util'], desc: 'Fetch JSON with error handling' });

// Get the code string of a template (or null if not found)
const code = BPI.Template.get('fetchHelper');

// Get the full metadata object { code, lang, tags, desc, saved }
const meta = BPI.Template.getMeta('fetchHelper');

// Apply a template to the editor (replaces all content)
BPI.Template.apply('fetchHelper');

// Insert at cursor (does not replace existing content)
BPI.Template.insert('fetchHelper');

// Save the current editor content as a template
BPI.Template.saveFromEditor('myBoilerplate', { lang: 'js', tags: ['starter'] });

// List template names — optionally filter by lang or tag
const allNames  = BPI.Template.list();
const jsNames   = BPI.Template.list({ lang: 'js' });
const utilNames = BPI.Template.list({ tag: 'util' });

// Rename a template
BPI.Template.rename('fetchHelper', 'jsonFetcher');

// Delete a template
BPI.Template.delete('jsonFetcher');

// Export all templates as a JSON string
const json = BPI.Template.export();
BPI.FS.writeFile('templates/backup.json', json);

// Import templates from a JSON string (merges with existing)
// Returns true on success, false on failure
BPI.Template.import(BPI.FS.readFile('templates/backup.json'));

// Clear all saved templates
BPI.Template.clear();
```

---

### BPI.I18n — Internationalization ★ New

Register per-plugin string dictionaries for full plugin localization. Active language follows `navigator.language` but can be overridden. Backed by `window.BeyoI18n_`.

```js
// Register translations for your plugin
BPI.I18n.register('MyPlugin', 'en', {
  greeting: 'Hello, {0}!',         // {0} = first placeholder
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

// Translate a key — uses current lang, falls back to 'en'
BPI.I18n.t('MyPlugin', 'greeting', ['Alice']);   // "Hello, Alice!" (or localized)
BPI.I18n.t('MyPlugin', 'error',    ['403']);     // "Something went wrong: 403"
BPI.I18n.t('MyPlugin', 'saved');                 // "File saved."

// Override the active language (persisted across sessions)
BPI.I18n.setLang('es');

// Get the current active language code
const lang = BPI.I18n.getLang();   // 'es'

// List all registered language codes across all plugins
const langs = BPI.I18n.available();   // ['en', 'es', 'hi']

// List all keys registered for a plugin + language
const keys = BPI.I18n.keys('MyPlugin', 'en');   // ['greeting', 'saved', 'error']

// Check if a key exists (checks current lang then falls back to en)
if (BPI.I18n.has('MyPlugin', 'greeting')) { /* ... */ }

// Dump all translations for a plugin to the IDE console
BPI.I18n.dump('MyPlugin');
```

---

## Extended Plugin Globals

These globals are available inside all plugin code alongside the `BPI.*` namespace.

---

### BeyoPanel_ — Floating Panels

Toggle-style floating panel (calling again with the same `id` closes it).

```js
BeyoPanel_({
  id:       'myDevPanel',
  title:    'Dev Tools',
  html:     '<p>Panel content here.</p>',
  position: 'bottom-right',   // 'bottom-right' | 'bottom-left' | 'top-right' | 'top-left' | 'center'
  width:    320,               // px (default 320)
  height:   240,               // max-height px (default 240)
  icon:     'ri-tools-line'    // RemixIcon class (default 'ri-plug-line')
});
// Call again with the same id to toggle off:
BeyoPanel_({ id: 'myDevPanel' });
```

---

### BeyoTheme_ — Live Theme Control

Legacy quick-apply shim for common CSS variables. Accepts a plain object.

```js
BeyoTheme_({
  primary: '#e879f9',   // --primary accent color
  bg:      '#0d0221',   // --bg-dark main background
  panel:   '#160420',   // --bg-panel sidebar/panel background
  editor:  '#1a0535',   // --bg-editor editor background
  border:  '#3b0764',   // --border color
  success: '#4ade80',   // --success color
  error:   '#f87171',   // --error color
  font:    16           // editor font size (number = px, or '16px' string)
});
```

All fields are optional. For full theme API (save/apply/list), use `BPI.Theme.*` instead.

---

### BeyoCommand_ — Named Commands

Register a named command with an optional keyboard shortcut.

```js
BeyoCommand_({
  name:     'My Command',
  shortcut: 'Ctrl+Shift+M',    // optional — fires action on keydown
  icon:     'ri-command-line', // optional
  action:   function() {
    BPI.Notify.success('Command executed!');
  }
});
```

---

### BeyoDialog_ — Native Dialogs

Show the IDE's native modal with custom content and action buttons. Action strings are evaluated as JavaScript.

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

A thin text strip pinned to the bottom center of the screen (distinct from `BPI.UI.statusChip`).

```js
// Show / update a status bar strip
BeyoStatusBar_({
  id:    'myStatus',          // optional stable id for updates
  text:  'Build: Running…',
  color: '#fbbf24',
  icon:  'ri-loader-4-line'   // RemixIcon class
});

// Update it later using the same id
BeyoStatusBar_({ id: 'myStatus', text: 'Build: Done ✓', color: '#4ade80' });
```

---

### BeyoWatch_ — File Open Watcher

Fires a callback whenever a matching file is opened in the editor (hooks into `onFileOpen`).

```js
// Watch for a specific file path
BeyoWatch_('src/app.js', function(path, content) {
  BPI.Console().log('app.js opened — ' + content.length + ' chars');
});

// Watch for a file extension (argument starts with '.')
BeyoWatch_('.py', function(path, content) {
  BPI.Notify.info('Python file opened: ' + path);
});
```

---

### BeyoContextMenu_ — Right-Click Items

Add a custom item to the file tree right-click context menu. Items are deduplicated by label.

```js
BeyoContextMenu_({
  label:  'Minify This File',
  icon:   'ri-scissors-line',
  filter: ['js', 'ts'],          // optional — only show for these extensions
  action: function(path) {       // path = right-clicked file path
    var code = BPI.FS.readFile(path);
    BPI.FS.writeFile(path, BPI.Format.minify(code));
    BPI.Notify.success('Minified: ' + path);
  }
});
```

---

### BeyoMobile_ — Mobile Helpers

Utilities for touch devices and the mobile dock.

```js
// Environment detection
BeyoMobile_.isTouch();    // true on touch devices
BeyoMobile_.isMobile();   // true if window.innerWidth ≤ 768

// Add a swipe gesture handler — direction: 'left' | 'right' | 'up' | 'down'
BeyoMobile_.addSwipeAction('right', function(dx) {
  BeyoMobile_.openSidebar();
});

// Add a button to the floating mobile dock (deduped by id)
BeyoMobile_.addDockButton({
  id:     'myDockBtn',
  icon:   'ri-magic-line',
  label:  'My Action',
  color:  '#a78bfa',
  action: function() { BPI.Notify.success('Dock button pressed!'); }
});

// Haptic feedback — 'light' (10ms) | 'medium' (30ms) | 'heavy' (60ms)
BeyoMobile_.haptic('medium');

// Set editor font size (accepts a number in px or a CSS string)
BeyoMobile_.fontScale(16);

// Open / close the file tree sidebar
BeyoMobile_.openSidebar();
BeyoMobile_.closeSidebar();

// Programmatically select a range of lines in the editor
BeyoMobile_.selectLines(5, 12);   // lines 5 through 12

// Show a line-picker dialog (mobile-friendly scrollable list)
BeyoMobile_.showLinePicker({
  title:    'Pick a line',
  multi:    true,              // allow range selection (default: true)
  onSelect: function(from, to) {
    BPI.Console().log('Selected lines ' + from + ' – ' + to);
  }
});
```

---

### BeyoFooterChip_ — Footer Chips

Full-config version of `add_footer` with an explicit remove API.

```js
// Add a chip to the plugin footer bar
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

Simple snippet registration (distinct from the full `BPI.Snippets` library). Snippets registered here live only in memory for the session.

```js
// Register a snippet with a trigger keyword
BeyoSnippet_({
  trigger:     'fetchjson',
  description: 'Async fetch helper',
  lang:        'js',
  code:        'async function fetchJSON(url) {\n  const r = await fetch(url);\n  return r.json();\n}'
});

// Insert a registered snippet at cursor position
BeyoInsertSnippet_('fetchjson');
```

---

## Plugin UI Helpers

These global helper functions are the standard way to inject UI into the IDE from plugin code.

### `add_footer(name, fn, opts?)`

Adds a clickable chip to the plugin footer bar. Calling again with the same name/id replaces the existing chip.

```js
// Minimal
add_footer('Format', function() { BPI.Format.formatCurrent(); });

// With options
add_footer('Lint', function() { BPI.Lint.lintCurrent(); }, {
  icon:    'ri-bug-line',      // RemixIcon class
  color:   '#f87171',          // chip text color
  tooltip: 'Lint this file',   // title attribute
  id:      'myLintChip'        // stable id for deduplication
});
```

### `add_header(name, fn, opts?)`

Adds a button to the IDE header toolbar. Buttons are tagged `data-plugin-btn` and removed when `afterBoot` re-fires.

```js
add_header('Deploy', function() { deploy(); }, {
  icon:    'ri-upload-cloud-line',
  tooltip: 'Deploy project',
  color:   '#4ade80',
  t:       'Deploy',   // optional visible text label next to the icon
  w:       80,         // optional button width (px)
  h:       32          // optional button height (px)
});
```

### `Toast_notification_({ message, type, duration? })`

Shows a toast notification. The `type` must be one of `'success'` | `'error'` | `'warn'` | `'info'`.

```js
Toast_notification_({ message: 'Done!',       type: 'success' });
Toast_notification_({ message: 'Watch out',   type: 'warn',   duration: 5000 });
Toast_notification_({ message: 'Bad things',  type: 'error',  duration: 8000 });
```

`Beyohints_` is an alias that accepts the same arguments.

---

## Complete Plugin Examples

### Example 1 — Auto-Save Git Snapshot

```js
// Commits the editor every 5 minutes automatically
OnThe_Function('afterBoot', function() {
  BPI.Logger.info('AutoSnapshot', 'Plugin started');

  setInterval(function() {
    var file = BPI.Editor().getActiveFile();
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
// Live word/line count displayed in the IDE status bar
OnThe_Function('afterBoot', function() {
  function update() {
    var editor = BPI.Editor();
    var code = editor ? editor.getValue() : '';
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
// Fetch any GitHub repo's README into a project file
OnThe_Function('afterBoot', function() {
  add_footer('Fetch README', async function() {
    var repo = await BPI.Notify.prompt('GitHub repo (owner/name):', 'octocat/Hello-World');
    if (!repo) return;
    var res = await BPI.Http.get('https://raw.githubusercontent.com/' + repo + '/HEAD/README.md');
    if (!res.ok) { BPI.Notify.error('Fetch failed: ' + res.status); return; }
    BPI.FS.writeFile('fetched-README.md', res.data);
    BPI.Notify.success('Saved as fetched-README.md');
  }, { icon: 'ri-github-line' });
});
```

---

### Example 4 — Keyboard Shortcut Suite

```js
// Register a set of useful shortcuts as a bundle
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
// Smart Formatter — remembers indent preference across sessions
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

**Plugin A — Publisher:**
```js
OnThe_Function('afterBoot', function() {
  add_footer('Emit File Info', function() {
    var data = { ts: Date.now(), file: BPI.Editor().getActiveFile() };
    BPI.Event.emit('sharedTools:fileInfo', data);
    BPI.Notify.info('Data emitted!');
  });
});
```

**Plugin B — Subscriber:**
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
// Apply Synthwave theme automatically when a .css file is opened
OnThe_Function('afterBoot', function() {
  BeyoWatch_('.css', function(path) {
    BeyoTheme_({ primary: '#e879f9', bg: '#0d0221', panel: '#160420' });
    BPI.Notify.info('Synthwave theme — editing: ' + path);
  });
});
```

---

### Example 8 — Custom Right-Click Menu Item

```js
// Add "Minify" to the file tree right-click menu (JS and TS files only)
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
// Save, apply, and cycle between named IDE themes
var THEMES = {
  'Default':   { '--primary':'#3b82f6','--bg-dark':'#09090b','--bg-panel':'#18181b','--bg-editor':'#282c34' },
  'Synthwave': { '--primary':'#e879f9','--bg-dark':'#0d0221','--bg-panel':'#160420','--bg-editor':'#1a0535' },
  'Forest':    { '--primary':'#4ade80','--bg-dark':'#071207','--bg-panel':'#0c1f0c','--bg-editor':'#102010' },
  'Ocean':     { '--primary':'#22d3ee','--bg-dark':'#030f1a','--bg-panel':'#061624','--bg-editor':'#091c2e' },
  'Rose':      { '--primary':'#f43f5e','--bg-dark':'#0f0609','--bg-panel':'#1a0b0f','--bg-editor':'#200d12' }
};

OnThe_Function('afterBoot', async function() {
  Object.entries(THEMES).forEach(function(kv) { BPI.Theme.save(kv[0], kv[1]); });

  add_footer('🎨 Theme', async function() {
    var names = Object.keys(THEMES).join(' | ');
    var choice = await BPI.Notify.prompt('Choose theme:', names);
    if (choice && THEMES[choice]) {
      BPI.Theme.apply(choice);
      BPI.Config.write('ThemeManager', 'active', choice);
      BPI.Notify.success('Theme: ' + choice);
    }
  }, { icon: 'ri-palette-line', color: '#a78bfa' });

  var last = BPI.Config.read('ThemeManager', 'active', null);
  if (last && THEMES[last]) BPI.Theme.apply(last);
});
```

---

### Example 10 — Web Worker Linter

```js
// Runs a linting scan in a Web Worker so the UI stays fully responsive
OnThe_Function('afterBoot', function() {
  add_footer('⚡ Worker Lint', async function() {
    BPI.UI.loadingOverlay(true, 'Scanning in background…');
    try {
      var code = BPI.Editor().getValue();
      var issues = await BPI.Worker.run(function(src) {
        var lines = src.split('\n');
        var found = [];
        lines.forEach(function(line, i) {
          if (/\beval\b/.test(line))      found.push({ line: i+1, msg: 'eval() used' });
          if (/\bvar\s/.test(line))       found.push({ line: i+1, msg: 'var (use let/const)' });
          if (/console\.log/.test(line))  found.push({ line: i+1, msg: 'console.log left in' });
          if (/TODO|FIXME/.test(line))    found.push({ line: i+1, msg: 'TODO/FIXME comment' });
          if (line.length > 200)          found.push({ line: i+1, msg: 'Line too long' });
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

### Example 11 — Localized i18n Plugin

```js
// Plugin UI adapts to the user's browser language automatically
var P = 'WeatherPlugin';
BPI.I18n.register(P, 'en', { fetch: 'Fetch Weather', done: 'Weather loaded!', fail: 'Request failed.' });
BPI.I18n.register(P, 'es', { fetch: 'Obtener Clima', done: '¡Clima cargado!', fail: 'Solicitud fallida.' });
BPI.I18n.register(P, 'hi', { fetch: 'मौसम लें',      done: 'मौसम लोड हुआ!',   fail: 'अनुरोध विफल।' });

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

### Example 12 — Storage-Backed Notes Plugin

```js
// Quick notes that persist across sessions via IndexedDB
var NOTE_KEY = 'quickNotes:content';

OnThe_Function('afterBoot', async function() {
  // Pre-load note on boot
  var saved = await BPI.Storage.getIDB(NOTE_KEY, '');

  add_footer('📝 Notes', async function() {
    var current = await BPI.Storage.getIDB(NOTE_KEY, '');
    var updated = await BPI.Notify.prompt('Quick Notes (saved automatically):', current || '');
    if (updated === null) return;   // cancelled
    await BPI.Storage.setIDB(NOTE_KEY, updated);
    BPI.FS.writeFile('notes/quick-notes.md', updated);
    BPI.Notify.success('Note saved to IDB + project!');
  }, { icon: 'ri-sticky-note-line', color: '#fbbf24' });

  BPI.Storage.getIDB(NOTE_KEY).then(function(n) {
    if (n) BPI.UI.statusChip({
      id:    'noteCount',
      text:  n.length + 'c note',
      color: '#fbbf24',
      icon:  'ri-sticky-note-line'
    });
  }).catch(function(){});
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
// Bad — appears in the browser DevTools console, not the IDE console
console.log('my debug info');

// Good — queryable, tagged, appears in the IDE console panel
BPI.Logger.debug('MyPlugin', 'my debug info');
```

**3. `.error()` is now correct — `.err()` is the legacy alias**
```js
// Both work in v1.15 (patched). Prefer .error() for readability.
BPI.Console().error('Something failed');
BPI.Console().err('Also works');
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
// Bad — anyone who opens the plugin editor can read this
const token = 'ghp_abc123xyz';

// Good — stored separately from plugin code
const token = BPI.Env.get('GITHUB_TOKEN');
```

**6. Use the correct BPI.FS method names**

| ❌ Wrong | ✅ Correct |
|---|---|
| `BPI.FS.list()` | `BPI.FS.listFiles()` |
| `BPI.FS.exists(p)` | `BPI.FS.fileExists(p)` |
| `BPI.FS.copy(a,b)` | `BPI.FS.copyFile(a,b)` |
| `BPI.FS.rename(a,b)` | `BPI.FS.renameFile(a,b)` |
| `BPI.FS.delete(p)` | `BPI.FS.deleteFile(p)` |
| `BPI.FS.size(p)` | `BPI.FS.getFileInfo(p).size` |

**7. Export plugins for backup and sharing**

In the Plugin Manager, use the **Export** button to download as `.json`. Use **Import** to load it on another device or share with teammates.

**8. Use `BPI.Theme.patch()` instead of `document.documentElement.style.setProperty()` directly**
```js
// Bad — bypasses BPI tracking, not reversible via BPI.Theme.reset()
document.documentElement.style.setProperty('--primary', '#f00');

// Good — tracked, reversible, compatible with BPI.Theme.save/apply
BPI.Theme.setVar('--primary', '#f00');
// Or bulk:
BPI.Theme.patch({ '--primary': '#f00', '--bg-editor': '#200' });
```

**9. Use `BPI.Worker.run()` for CPU-heavy operations**
```js
// Bad — freezes the IDE UI while running
const result = myExpensiveSort(largeArray);

// Good — runs in a background thread, UI stays fully responsive
const result = await BPI.Worker.run(function(arr) {
  return arr.sort(function(a, b) { return a - b; });
}, largeArray);
```

**10. Use `BPI.Storage.setIDB()` for large or complex data**
```js
// Bad — localStorage has a 5MB limit and blocks the main thread on large parse
localStorage.setItem('myPlugin:bigData', JSON.stringify(hugeArray));

// Good — IndexedDB handles large data asynchronously, no size issues
await BPI.Storage.setIDB('myPlugin:bigData', hugeArray);
```

**11. Register i18n translations at the top of your plugin, before any UI**
```js
// Good — translations registered before add_footer() calls
var P = 'MyPlugin';
BPI.I18n.register(P, 'en', { run: 'Run', cancel: 'Cancel' });
BPI.I18n.register(P, 'es', { run: 'Ejecutar', cancel: 'Cancelar' });

OnThe_Function('afterBoot', function() {
  add_footer(BPI.I18n.t(P, 'run'), runFn);
});
```

**12. Use `BPI.Color.isAccessible()` to verify custom UI colors pass WCAG AA**
```js
var myColor = '#a78bfa';
var bg      = BPI.Theme.getVar('--bg-dark');
if (!BPI.Color.isAccessible(myColor, bg)) {
  myColor = BPI.Color.lighten(myColor, 0.2);   // auto-fix
}
BPI.UI.statusChip({ id: 'myChip', text: 'OK', color: myColor });
```

**13. Profile expensive operations with `BPI.Bench`**
```js
BPI.Bench.start('myHeavyOp');
// ... expensive work ...
BPI.Bench.end('myHeavyOp');   // automatically logs timing to IDE console
```

---

## Changelog — v1.15 Patch Notes

These bugs were found during a full source audit and fixed in the patched HTML:

| # | Bug | Impact | Fix |
|---|---|---|---|
| 1 | `BeyoConsole_.error()` silently failed — method did not exist | **Critical** — every BPI module's error logs were swallowed | Added `.error` as a proper alias for `UI.appendConsole('error', …)` |
| 2 | `Toast_notification_({ duration })` parameter was silently ignored | Medium — custom durations had no effect | Passed `duration` through to `Toast.show()` |
| 3 | `BeyoConsole_.clear()` was documented but missing | Low | Added `.clear()` method that clears `#debugContent` |
| 4 | `BPI.FS.writeFile` / `deleteFile` / `renameFile` each fired `onFileSystemChange` without a re-entrancy guard, causing stack overflows if BPI.FS was called inside the hook | **Critical** | Added `_fsHookRunning` guard to all three methods and `moveFile` |
| 5 | `BPI.Project.switchTo()` called `triggerHook('afterBoot')` directly, causing infinite recursion if a plugin called `switchTo` inside `afterBoot` | **Critical** | Now calls `window.Project.switch()` instead |
| 6 | `BeyoContextMenu_._confirmDelete()` used `confirm()` which breaks in PWA/standalone mode and failed on snippet names with quotes | Medium | Replaced with `Dlg.show()` |

## New in v1.15 — API Additions

| Module | Methods Added |
|---|---|
| `BPI.Theme` | `setVar`, `getVar`, `save`, `apply`, `list`, `delete`, `patch`, `reset`, `current`, `isDark`, `defaults` |
| `BPI.Color` | `hexToRgb`, `rgbToHex`, `mix`, `lighten`, `darken`, `luminance`, `contrast`, `isAccessible`, `readable`, `toRgba`, `invert`, `randomHex` |
| `BPI.Storage` | `setIDB`, `getIDB`, `delIDB`, `keysIDB`, `clearIDB` · `session.set/get/del/keys/has/clear` · `local.set/get/del/keys/has/clear` · `exportLocal`, `importLocal` |
| `BPI.Worker` | `spawn`, `post`, `onMessage`, `terminate`, `terminateAll`, `list`, `run(fn, data, timeoutMs)` |
| `BPI.Template` | `save`, `get`, `getMeta`, `apply`, `insert`, `saveFromEditor`, `list`, `rename`, `delete`, `export`, `import`, `clear` |
| `BPI.I18n` | `register`, `t`, `setLang`, `getLang`, `available`, `keys`, `has`, `dump` |
| `BPI.FS` | `readJson`, `writeJson`, `appendFile`, `moveFile`, `getStats`, `readAll`, `lineCount`, `listByExt` |
| `BPI.Execute` | `runFile`, `interval`, `retry`, `debounce`, `throttle` |
| `BPI.Http` | `put`, `patch`, `delete`, `loadCSS`, `getJSON` |
| `BPI.Format` | `toPascalCase`, `toKebabCase`, `htmlEncode`, `htmlDecode`, `truncate`, `stripHtml`, `wrapLines`, `countOccurrences`, `padLines`, `toTable` |
| `BPI.UI` | `tooltip`, `contextMenu`, `badgeCount`, `highlight`, `loadingOverlay` |
| `BPI.Notify` | `progress`, `desktop`, `queue` |
| `BPI.Logger` | `table`, `groupStart`, `groupAdd`, `groupEnd` |
| `BPI.Snippets` | `getCode`, `saveFromEditor`, `copy`, `rename`, `import`, `export`, `showManager`, `_raw` |

---

## Getting Help

| Resource | Link |
|---|---|
| 📧 Email Support | [help.magmamine@gmail.com](mailto:help.magmamine@gmail.com) |
| 💬 Discord | [discord.gg/MVKGsBgmkm](https://discord.gg/MVKGsBgmkm) |
| 🌐 Official Website | [beyoneer.xyz](https://beyoneer.xyz) |
| 📟 In-IDE Reference | Run `BPI.help()` in the Beyoneer console |
| 💻 Terminal shortcut | Type `bpi help` in the IDE terminal |

---

*BPI Documentation · Beyoneer IDE v1.15 (Patched & Source-Verified) · By **MagmaMinesTeam***  
*© MagmaMinesTeam — [help.magmamine@gmail.com](mailto:help.magmamine@gmail.com) — [beyoneer.xyz](https://beyoneer.xyz)*
