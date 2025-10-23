# Tools Guide: Building Experimental Web Tools

A guide for building systematic, experimental web tools in the toolshed repository. Inspired by Simon Willison's [tools.simonwillison.net](https://github.com/simonw/tools), but adapted for playful experiments and learning with coding agents.

---

## 1. Philosophy & Intent

**Toolshed** is a curated collection of experimental web utilities with these characteristics:

- **Experimental first**: Tools are learning projects, not production systems
- **Systematic approach**: Proven patterns and structure from established projects (Simon Willison)
- **Playful & humorous**: Room for creative, unconventional projects alongside practical tools
- **Agent-friendly**: Designed to be built collaboratively with Claude Code
- **Minimal dependencies**: Client-side only, self-contained HTML files
- **Transparent**: Honest postmortems about what works and what doesn't
- **Zero friction**: No build step, deploy directly to GitHub Pages

Each tool should have a clear purpose, but the journey (failures, iterations, learnings) is equally important as the destination.

---

## 2. Repository Structure

```
/toolshed/
├── *.html                    # Individual tool files
├── *.docs.md                 # Documentation for each tool
├── index.html                # Landing page listing all tools
├── DEPLOYMENT_NOTES.md       # How deployment works
├── TOOLS_GUIDE.md            # This file
├── *_POSTMORTEM.md           # Project retrospectives (optional)
├── tests/
│   └── test_*.py            # Pytest/Playwright tests (future)
├── .github/workflows/
│   └── deploy.yml           # Deploys to GitHub Pages
└── .gitignore
```

**Key differences from Simon's setup:**
- Simpler file structure (no build scripts yet, can add as you scale)
- Postmortems are optional but encouraged for learning
- Testing framework available but not required initially
- Focus on agent collaboration patterns

---

## 3. Tool File Naming Convention

### HTML Files
- **Format**: `{tool-name}.html` (lowercase, hyphens only)
- **Examples**: `query-string-stripper.html`, `youtube-to-mp3.html`, `json-formatter.html`

### Documentation Files
- **Format**: `{tool-name}.docs.md`
- **Purpose**: Brief description of what the tool does
- **Placement**: Same directory as the HTML file

### Branch Names
- **Format**: `{tool-name}-experiment` or `feature/{tool-name}`
- **Examples**: `json-formatter-experiment`, `feature/clipboard-tools`

### Postmortem Files (Optional)
- **Format**: `{TOOL_NAME}_POSTMORTEM.md` (uppercase)
- **When to write**: After completing a tool, especially if it failed or taught lessons
- **Content**: What was learned, what failed, alternatives tried, and why

---

## 4. Documentation: docs.md Template

Each tool should have a corresponding `.docs.md` file:

```markdown
# Tool Name

Brief description of what the tool does in 1-2 sentences. Include the main value proposition or use case.

## Features

- Feature 1
- Feature 2
- Feature 3

## How to use

Step-by-step instructions or workflow description.

## Technical notes (optional)

Any interesting technical decisions, limitations, or implementation details worth noting.

<!-- Generated from commit: [auto-filled by build process if applicable] -->
```

**Minimal example:**
```markdown
Query string remover - quickly strip query parameters from URLs. Paste a URL, click to copy the cleaned version.
```

---

## 5. HTML Structure Patterns

### Pattern 1: Simple Stateless Tool
**Use when**: Processing input in real-time without external dependencies
**Examples**: `query-string-stripper.html`, `escape-entities.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tool Name</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
                         Helvetica, Arial, sans-serif;
            max-width: 900px;
            margin: 0 auto;
            padding: 20px;
            line-height: 1.6;
        }

        h1 {
            margin-top: 0;
            font-size: clamp(24px, 5vw, 32px);
        }

        /* Mobile-first: stack vertically */
        @media (max-width: 600px) {
            body {
                padding: 12px;
            }
            h1 {
                font-size: 24px;
            }
        }

        /* Tablet and up: can use 2-column layouts */
        @media (min-width: 768px) {
            .container {
                display: grid;
                grid-template-columns: 1fr 1fr;
                gap: 20px;
            }
        }
    </style>
</head>
<body>
    <h1>Tool Name</h1>
    <p>Brief description of what this tool does</p>

    <textarea id="input" placeholder="Paste content here..."></textarea>
    <button id="process">Process</button>

    <div id="output" style="display: none;">
        <textarea id="result" readonly></textarea>
        <button id="copy">Copy to clipboard</button>
    </div>

    <div id="error" class="error" style="display: none;"></div>

    <script>
        const inputEl = document.getElementById('input');
        const processBtn = document.getElementById('process');
        const outputEl = document.getElementById('output');
        const resultEl = document.getElementById('result');
        const errorEl = document.getElementById('error');
        const copyBtn = document.getElementById('copy');

        function hideError() {
            errorEl.style.display = 'none';
        }

        function showError(message) {
            errorEl.textContent = message;
            errorEl.style.display = 'block';
        }

        function processInput() {
            hideError();
            const input = inputEl.value.trim();

            if (!input) {
                showError('Please enter some content');
                outputEl.style.display = 'none';
                return;
            }

            try {
                // Your processing logic here
                const result = input.toUpperCase(); // Example
                resultEl.value = result;
                outputEl.style.display = 'block';
            } catch (err) {
                showError(`Error: ${err.message}`);
                outputEl.style.display = 'none';
            }
        }

        processBtn.addEventListener('click', processInput);
        inputEl.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && e.ctrlKey) {
                processInput();
            }
        });

        copyBtn.addEventListener('click', () => {
            navigator.clipboard.writeText(resultEl.value).then(() => {
                const originalText = copyBtn.textContent;
                copyBtn.textContent = 'Copied!';
                setTimeout(() => {
                    copyBtn.textContent = originalText;
                }, 2000);
            });
        });
    </script>
</body>
</html>
```

### Pattern 2: Tool with External APIs or Libraries
**Use when**: Fetching data from external sources or using CDN libraries
**Examples**: `json-extractor.html`, `markdown-viewer.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tool with API</title>
    <!-- Load library from CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/marked/11.1.1/marked.min.js"></script>
    <style>
        /* ... base styles ... */
        button:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }
        .loading::after {
            content: ' ⟳';
            animation: spin 1s infinite;
        }
        @keyframes spin {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }
    </style>
</head>
<body>
    <h1>Tool with API</h1>
    <input type="text" id="search" placeholder="Enter search term...">
    <button id="fetch">Fetch</button>
    <div id="results"></div>
    <div id="error" style="display: none;"></div>

    <script>
        const fetchBtn = document.getElementById('fetch');
        const searchInput = document.getElementById('search');
        const resultsDiv = document.getElementById('results');
        const errorDiv = document.getElementById('error');

        async function fetchData() {
            const query = searchInput.value.trim();
            if (!query) return;

            fetchBtn.disabled = true;
            fetchBtn.classList.add('loading');
            errorDiv.style.display = 'none';

            try {
                const response = await fetch(`https://api.example.com/search?q=${query}`);
                if (!response.ok) throw new Error(`HTTP ${response.status}`);

                const data = await response.json();
                resultsDiv.innerHTML = `<pre>${JSON.stringify(data, null, 2)}</pre>`;
            } catch (err) {
                errorDiv.textContent = `Error: ${err.message}`;
                errorDiv.style.display = 'block';
            } finally {
                fetchBtn.disabled = false;
                fetchBtn.classList.remove('loading');
            }
        }

        fetchBtn.addEventListener('click', fetchData);
        searchInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') fetchData();
        });
    </script>
</body>
</html>
```

---

## 6. Data Loading Patterns

### Pattern 1: Load from URL Hash
```javascript
// Load data from URL hash: ?tool=value or #gist=abc123
function loadFromHash() {
    const params = new URLSearchParams(window.location.hash.slice(1));
    const value = params.get('gist');

    if (value) {
        // Load gist from GitHub API
        fetch(`https://api.github.com/gists/${value}`)
            .then(r => r.json())
            .then(data => {
                document.getElementById('input').value = Object.values(data.files)[0].content;
            });
    }
}

window.addEventListener('hashchange', loadFromHash);
loadFromHash();
```

### Pattern 2: File Input & Drag-and-Drop
```javascript
const dropzone = document.getElementById('dropzone');
const fileInput = document.getElementById('fileInput');

dropzone.addEventListener('click', () => fileInput.click());

dropzone.addEventListener('dragover', (e) => {
    e.preventDefault();
    dropzone.classList.add('drag-active');
});

dropzone.addEventListener('dragleave', () => {
    dropzone.classList.remove('drag-active');
});

dropzone.addEventListener('drop', (e) => {
    e.preventDefault();
    dropzone.classList.remove('drag-active');
    handleFiles(e.dataTransfer.files);
});

fileInput.addEventListener('change', (e) => {
    handleFiles(e.target.files);
});

function handleFiles(files) {
    for (const file of files) {
        const reader = new FileReader();
        reader.onload = (e) => {
            processContent(e.target.result);
        };
        reader.readAsText(file);
    }
}
```

### Pattern 3: Clipboard Interaction
```javascript
// Read from clipboard
async function readClipboard() {
    try {
        const items = await navigator.clipboard.read();
        for (const item of items) {
            if (item.types.includes('text/plain')) {
                const text = await item.getType('text/plain').text();
                processContent(text);
            }
        }
    } catch (err) {
        console.error('Clipboard read failed:', err);
    }
}

// Write to clipboard
async function copyToClipboard(text) {
    try {
        await navigator.clipboard.writeText(text);
        return true;
    } catch (err) {
        // Fallback for older browsers
        const textarea = document.createElement('textarea');
        textarea.value = text;
        document.body.appendChild(textarea);
        textarea.select();
        document.execCommand('copy');
        document.body.removeChild(textarea);
        return true;
    }
}
```

---

## 7. UI Component Patterns

### Copy to Clipboard Button
Standard pattern used across tools:

```javascript
const copyBtn = document.getElementById('copy-button');
const textEl = document.getElementById('text-to-copy');

copyBtn.addEventListener('click', async () => {
    try {
        await navigator.clipboard.writeText(textEl.value);
        const originalText = copyBtn.textContent;
        copyBtn.textContent = '✓ Copied!';
        copyBtn.disabled = true;

        setTimeout(() => {
            copyBtn.textContent = originalText;
            copyBtn.disabled = false;
        }, 2000);
    } catch (err) {
        console.error('Copy failed:', err);
    }
});
```

**CSS:**
```css
button {
    background-color: #0066cc;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
}

button:hover {
    background-color: #0052a3;
}

button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}
```

### Loading States
```javascript
async function loadData() {
    button.disabled = true;
    button.textContent = 'Loading...';

    try {
        const data = await fetchFromAPI();
        displayResults(data);
    } catch (err) {
        showError(err.message);
    } finally {
        button.disabled = false;
        button.textContent = 'Load Data';
    }
}
```

### Error Messages
```html
<div id="error" class="error"></div>

<style>
.error {
    color: #d32f2f;
    background-color: #ffebee;
    border-left: 4px solid #d32f2f;
    padding: 12px;
    margin: 12px 0;
    border-radius: 4px;
    display: none;
}

.error.visible {
    display: block;
}
</style>

<script>
function showError(message) {
    const errorEl = document.getElementById('error');
    errorEl.textContent = message;
    errorEl.classList.add('visible');
}

function hideError() {
    document.getElementById('error').classList.remove('visible');
}
</script>
```

---

## 8. Responsive Design Standards

### Breakpoints
- **Mobile**: < 600px (default/mobile-first)
- **Tablet**: 600px - 768px
- **Desktop**: ≥ 768px and up to 1200px
- **Large**: ≥ 1200px

### Mobile-First Approach
```css
/* Base: mobile styles */
body {
    font-size: 16px;
    padding: 12px;
}

.container {
    display: block;
}

/* Tablet and up */
@media (min-width: 768px) {
    body {
        padding: 20px;
    }
    .container {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 20px;
    }
}

/* Large screens */
@media (min-width: 1200px) {
    body {
        max-width: 1200px;
        margin: 0 auto;
        padding: 32px;
    }
}
```

### Fluid Typography
```css
/* Instead of fixed sizes, use clamp() for fluid scaling */
h1 {
    font-size: clamp(24px, 5vw, 42px);
}

h2 {
    font-size: clamp(20px, 4vw, 32px);
}

body {
    font-size: clamp(14px, 1.2vw, 18px);
}
```

### Flexible Layouts
```css
/* Flexbox for simple layouts */
.button-group {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
}

/* Grid for complex layouts */
.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 16px;
}

/* Always use border-box */
* {
    box-sizing: border-box;
}
```

---

## 9. External Libraries & CDN Loading

### Recommended CDNs
- **jsDelivr**: Fast, reliable, for modern JS/CSS (`https://cdn.jsdelivr.net/`)
- **cdnjs**: Large catalog of popular libraries (`https://cdnjs.cloudflare.com/`)
- **unpkg**: NPM packages directly (`https://unpkg.com/`)

### Common Libraries

```html
<!-- Markdown parsing -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/marked/11.1.1/marked.min.js"></script>

<!-- JSON formatting/validation -->
<script src="https://cdn.jsdelivr.net/npm/json5@2/dist/index.min.js"></script>

<!-- QR codes -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<!-- Color conversion -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/chroma-js/2.4.2/chroma.min.js"></script>

<!-- Chart/visualization -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<!-- Date/time manipulation -->
<script src="https://cdn.jsdelivr.net/npm/date-fns"></script>
```

### Loading Strategy
```html
<!-- For critical libraries, load synchronously -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/marked/11.1.1/marked.min.js"></script>

<!-- For non-critical, use async -->
<script async src="https://example.com/analytics.js"></script>

<!-- For ES modules -->
<script type="module">
    import Chart from 'https://cdn.jsdelivr.net/npm/chart.js/+esm';
</script>
```

---

## 10. Development Workflow with Claude Code

### Structuring Tool Experiments for Agents

When working with Claude Code to build tools, structure your request with these elements:

```markdown
## Tool Request Template

**Tool Name**: [Clear, descriptive name]

**Purpose**: One sentence describing what the tool does

**Core Functionality**:
- Feature 1
- Feature 2
- Feature 3

**Success Criteria**:
- Tool successfully [does X]
- Tool handles [edge case Y]
- Tool is responsive on mobile

**Constraints**:
- Client-side only (no backend)
- Vanilla JavaScript (no frameworks)
- Single HTML file
- < 500 lines of code (prefer simplicity)

**Inspiration/Reference**:
- Link to similar tool or pattern to follow
- Technical patterns to use
```

### Example Request
```markdown
## Build JSON Formatter Tool

**Purpose**: Convert JSON strings to formatted, highlighted output

**Core Functionality**:
- Accept pasted JSON
- Detect and report invalid JSON
- Format with 2-space indentation
- Show statistics (keys, nesting depth)
- Copy formatted output to clipboard

**Success Criteria**:
- Handles valid and invalid JSON gracefully
- Mobile responsive
- Fast (no lag with 10KB+ JSON)

**Constraints**:
- Client-side only
- Use marked.js for syntax highlighting if needed
- Follow simple-tool pattern
```

### Branch Naming for Experiments
- Feature branch: `feature/{tool-name}` or `{tool-name}-experiment`
- Create focused, single-tool branches
- Write clear commit messages
- Include postmortem before merging (if lessons learned)

---

## 11. Testing Structure (Future)

As toolshed grows, add Playwright + pytest tests:

```python
# tests/test_example_tool.py
import pytest
from playwright.sync_api import Page, expect

@pytest.fixture(scope="module")
def local_server():
    # Start http server for testing
    pass

def test_initial_state(page: Page):
    page.goto("http://127.0.0.1:8000/example-tool.html")
    expect(page.locator("h1")).to_have_text("Tool Name")

def test_process_input(page: Page):
    page.goto("http://127.0.0.1:8000/example-tool.html")
    page.fill("#input", "test data")
    page.click("#process")
    expect(page.locator("#output")).to_be_visible()
```

**Run locally:**
```bash
# Start server
python -m http.server 8000

# Run tests
pytest -v
```

---

## 12. Build & Deployment

Toolshed uses GitHub Pages with automatic deployment via GitHub Actions.

### Deployment Workflow
1. **Push to main branch**: Automatic GitHub Actions workflow triggers
2. **Deploy to GitHub Pages**: Site updates at `https://rpasetes.github.io/toolshed/`
3. **Update index.html**: Link new tools in landing page

### Manual Server for Development
```bash
# Start local development server
python -m http.server 8000

# Visit http://localhost:8000/tool-name.html
```

---

## 13. Documentation & Postmortems

### When to Write a Postmortem
- Tool doesn't work as planned
- Significant learnings from the experiment
- API/service limitations discovered
- Interesting technical challenges

### Postmortem Template
```markdown
# {TOOL_NAME} Postmortem

## What We Built
Brief description of the tool and goals.

## What Worked
- Thing 1
- Thing 2

## What Didn't Work
- Thing 3
- Thing 4

## Why It Failed
Technical analysis of blockers, API limitations, browser constraints, etc.

## Lessons Learned
- Learning 1
- Learning 2

## Alternatives & Next Steps
If building again, what would you try instead?

## Time Spent
Rough estimate of development time.
```

---

## 14. Checklist for New Tools

- [ ] Create `{tool-name}.html` file
- [ ] Create `{tool-name}.docs.md` with description
- [ ] Implement core functionality with error handling
- [ ] Make responsive (test on mobile)
- [ ] Add copy-to-clipboard or export functionality (if applicable)
- [ ] Test with edge cases and invalid input
- [ ] Update `index.html` with link to new tool
- [ ] Push to feature branch, create PR
- [ ] Merge and verify on live site
- [ ] Write postmortem if there are learnings (optional)

---

## 15. Key File References

| Pattern | File Example | Size | Complexity |
|---------|------|------|-----------|
| Minimal tool | `query-string-stripper.html` | ~150 lines | ⭐ |
| Simple processor | `escape-entities.html` | ~200 lines | ⭐ |
| With external API | `json-string-extractor.html` | ~400 lines | ⭐⭐ |
| With visualization | `mask-visualizer.html` | ~500 lines | ⭐⭐ |

---

## 16. Quick Start: Building Your First Tool

1. **Pick a tool idea** - Something small and focused
2. **Copy the minimal pattern** from section 5
3. **Customize HTML/CSS** for your specific tool
4. **Implement core logic** in JavaScript
5. **Test locally** with `python -m http.server 8000`
6. **Add error handling** for edge cases
7. **Make responsive** - test on mobile
8. **Create docs.md** with brief description
9. **Update index.html** with link
10. **Push and deploy** - GitHub Actions handles the rest

---

## 17. Philosophy Reminders

- **Start simple**: Single-purpose tools > Swiss Army knives
- **Learn in public**: Document what you discover
- **Embrace failure**: Postmortems are valuable
- **Mobile first**: Design for phones, enhance for desktop
- **No magic**: Vanilla JS, minimal dependencies
- **Agent-friendly**: Clear specs make Claude more helpful
- **Have fun**: Experimental means trying unusual ideas

---

## 18. Claude Code Setup (Local)

To get the best experience working with Claude Code on toolshed tools, set up local instructions that give the agent context about your patterns and philosophy.

### Setup Instructions

1. **Create the `.claude` directory** (if it doesn't exist):
   ```bash
   mkdir -p .claude
   ```

2. **Create `.claude/instructions.md`** with toolshed context:
   ```markdown
   # Toolshed Development Context

   When working on tools in this repository, refer to TOOLS_GUIDE.md for:
   - Tool file naming conventions
   - HTML structure patterns
   - Responsive design standards
   - UI component patterns
   - Data loading patterns
   - Deployment workflow

   Key principles:
   - Client-side only, self-contained HTML files
   - Mobile-first responsive design
   - Vanilla JavaScript (no frameworks)
   - Tools hosted on GitHub Pages at https://rpasetes.github.io/toolshed/
   - Each tool gets: {tool-name}.html + {tool-name}.docs.md
   - Honest postmortems for learnings

   For new tool requests, use the template in TOOLS_GUIDE.md section 10.
   ```

3. **`.claude/` is gitignored** - This directory stays local and isn't committed to the repo. It's personal configuration for your Claude Code setup.

### Why This Works

- Claude Code automatically reads `.claude/instructions.md` in future conversations
- You get consistent context without polluting the repo
- Different collaborators can have their own setup
- Configuration stays in sync with your TOOLS_GUIDE.md

---

## Additional Resources

- [Simon Willison's Tools Repo](https://github.com/simonw/tools)
- [MDN: Web Standards](https://developer.mozilla.org/)
- [Can I Use](https://caniuse.com/) - Browser compatibility
- [jsDelivr](https://www.jsdelivr.com/) - CDN search
