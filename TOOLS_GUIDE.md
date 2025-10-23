# Tools Guide: Technical Patterns & Standards

A practical guide for building client-side web tools in the toolshed repository. Patterns and standards derived from Simon Willison's [tools.simonwillison.net](https://github.com/simonw/tools).

---

## 1. Repository Structure

```
/toolshed/
├── *.html              # Individual tool files
├── *.docs.md           # Tool documentation with gist links
├── index.html          # Landing page listing all tools
├── TOOLS_GUIDE.md      # This file (technical patterns)
├── .gitignore
└── .github/workflows/
    └── deploy.yml     # Automatic GitHub Pages deployment
```

**Key principles:**
- Single HTML file per tool (self-contained, no build step)
- No external dependencies except CDN libraries
- Client-side only processing
- Vanilla JavaScript (no frameworks)
- Mobile-responsive design required

---

## 2. Tool File Naming & Documentation

### HTML Files
- **Format**: `{tool-name}.html` (lowercase, hyphens only)
- **Examples**: `pomodoro.html`, `query-string-stripper.html`

### Documentation Files
- **Format**: `{tool-name}.docs.md`
- **Content**: Brief description of tool + links to external documentation (gists, etc.)

### Git Branches
- **Format**: `feature/{tool-name}`
- **Examples**: `feature/pomodoro-timer`, `feature/json-formatter`

### Example docs.md
```markdown
# Tool Name

Brief 1-2 sentence description of what the tool does and its primary use case.

## Features

- Feature 1
- Feature 2
- Feature 3

## How to use

Step-by-step instructions or workflow description.

---

**Development notes**: [GitHub Gist Link](https://gist.github.com/...)
```

---

## 3. HTML Structure Patterns

### Basic Tool Template
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tool Name</title>
    <style>
        * { box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
                         Helvetica, Arial, sans-serif;
            max-width: 900px;
            margin: 0 auto;
            padding: 20px;
            line-height: 1.6;
        }
        @media (max-width: 600px) {
            body { padding: 12px; }
        }
    </style>
</head>
<body>
    <h1>Tool Name</h1>
    <p>Brief description</p>

    <!-- Tool UI here -->

    <script>
        // Tool logic here
    </script>
</body>
</html>
```

### Key Requirements
- Responsive `<meta name="viewport">` tag
- Universal `box-sizing: border-box`
- Max-width centered container (600-1200px)
- Mobile media query at 600px minimum
- All CSS and JavaScript inline (no external files)

---

## 4. Responsive Design Standards

### Mobile-First Approach
Base styles target mobile, enhance with media queries:

```css
/* Mobile first */
body { font-size: 16px; padding: 12px; }
.container { display: block; }

/* Tablet (600px+) */
@media (min-width: 600px) {
    body { padding: 20px; }
}

/* Desktop (768px+) */
@media (min-width: 768px) {
    .container { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
}

/* Large (1200px+) */
@media (min-width: 1200px) {
    body { max-width: 1200px; margin: 0 auto; }
}
```

### Fluid Typography
```css
/* Use clamp() instead of fixed sizes */
h1 { font-size: clamp(24px, 5vw, 42px); }
body { font-size: clamp(14px, 1.2vw, 18px); }
```

### Layout Patterns
```css
/* Flexbox for simple layouts */
.button-group {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
}

/* Grid for responsive multi-column */
.grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 16px;
}
```

---

## 5. Data Loading & Input Patterns

### URL Hash Parameters
```javascript
// Load data from URL hash: ?tool=value or #gist=abc123
function loadFromHash() {
    const params = new URLSearchParams(window.location.hash.slice(1));
    const value = params.get('gist');
    if (value) {
        fetch(`https://api.github.com/gists/${value}`)
            .then(r => r.json())
            .then(data => processData(data));
    }
}

window.addEventListener('hashchange', loadFromHash);
loadFromHash();
```

### File Input with Drag & Drop
```javascript
const dropzone = document.getElementById('dropzone');
const fileInput = document.getElementById('fileInput');

dropzone.addEventListener('click', () => fileInput.click());
dropzone.addEventListener('dragover', e => {
    e.preventDefault();
    dropzone.classList.add('drag-active');
});
dropzone.addEventListener('drop', e => {
    e.preventDefault();
    dropzone.classList.remove('drag-active');
    handleFiles(e.dataTransfer.files);
});

fileInput.addEventListener('change', e => handleFiles(e.target.files));
```

### Clipboard Interaction
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

## 6. UI Component Patterns

### Copy to Clipboard Button
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
.error.visible { display: block; }
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

## 7. External Libraries & CDN Loading

### Recommended CDNs
- **jsDelivr**: Fast, npm packages + project files (`https://cdn.jsdelivr.net/`)
- **cdnjs**: Large catalog of popular libraries (`https://cdnjs.cloudflare.com/`)
- **unpkg**: NPM packages directly (`https://unpkg.com/`)

### Common Libraries
```html
<!-- Markdown parsing -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/marked/11.1.1/marked.min.js"></script>

<!-- JSON formatting -->
<script src="https://cdn.jsdelivr.net/npm/json5@2/dist/index.min.js"></script>

<!-- QR codes -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<!-- Color manipulation -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/chroma-js/2.4.2/chroma.min.js"></script>

<!-- Charts/visualization -->
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

## 8. Development Workflow

### Creating a New Tool

1. **Create feature branch**: `git checkout -b feature/{tool-name}`
2. **Build tool**: Create `{tool-name}.html` using template from section 3
3. **Test locally**: `python -m http.server 8000` → `http://localhost:8000/{tool-name}.html`
4. **Make responsive**: Test on mobile (600px breakpoint minimum)
5. **Create docs**: Add `{tool-name}.docs.md` with description
6. **Update index**: Add link to tool on `index.html`
7. **Test all**: Verify tool works and is responsive
8. **Commit**: Clear commit message
9. **Push**: `git push -u origin feature/{tool-name}`
10. **Create PR**: Include gist links if applicable

### Commit Message Format
```
Add {tool-name} tool with {brief description}

{Detailed description of what was built and why}

If applicable:
- Development notes: [GitHub Gist Link](https://gist.github.com/...)
- Reference: [Other link](...)

🤖 Generated with Claude Code

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## 9. Deployment

Tools are automatically deployed to GitHub Pages when merged to main.

**Workflow:**
1. Feature branch → PR → Review → Merge to main
2. GitHub Actions workflow triggers automatically
3. Site updates at `https://rpasetes.github.io/toolshed/`
4. No manual deployment steps needed

---

## 10. References & Session Notes

Session transcripts and development notes for tools are documented as GitHub Gists and linked from individual tool `.docs.md` files.

### Repository Session Notes

- **Deployment Notes (Oct 21, 2025)**: [GitHub Gist](https://gist.github.com/rpasetes/4ac0594e0a9d35125778a7c22d010b30) - GitHub Pages deployment configuration
- **Toolshed Foundation Session (Oct 22, 2025)**: [Full Transcript](https://gist.github.com/rpasetes/7449a90946ed876efd87c4bb94aac92f) - Documents the creation of TOOLS_GUIDE.md and pomodoro timer tool

---

## Checklist for New Tools

- [ ] Create `{tool-name}.html` file
- [ ] Create `{tool-name}.docs.md` with description
- [ ] Implement core functionality
- [ ] Test locally with `python -m http.server 8000`
- [ ] Make responsive (test at 600px breakpoint)
- [ ] Add error handling and validation
- [ ] Test with edge cases and invalid input
- [ ] Update `index.html` with tool link
- [ ] Commit with clear message (include gist links if applicable)
- [ ] Push to feature branch and create PR
- [ ] Merge when approved

---

## Quick Reference: Common Patterns

| Pattern | Use Case |
|---------|----------|
| Simple stateless tool | Input → Process → Output (no state) |
| Tool with external API | Fetch data from API, display results |
| Tool with file handling | Accept file input or drag-drop |
| Tool with clipboard | Read from or write to clipboard |
| Tool with visualization | Charts, graphics, or canvas rendering |

---

## Additional Resources

- [Simon Willison's Tools Repo](https://github.com/simonw/tools)
- [MDN: Web Standards](https://developer.mozilla.org/)
- [Can I Use](https://caniuse.com/) - Browser compatibility
- [jsDelivr](https://www.jsdelivr.com/) - CDN library search
