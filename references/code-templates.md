# Code templates

Essential patterns for browser extensions. Claude can generate standard boilerplate; this file covers non-obvious patterns and gotchas.

## Logging requirements

All console logs must be prefixed with the extension name for easy filtering:

```javascript
const EXT_NAME = 'MyExtension';

function log(message, type = 'info') {
  const prefix = `[${EXT_NAME}]`;
  const styles = {
    info: 'color: #2196F3',
    success: 'color: #4CAF50',
    warning: 'color: #FF9800',
    error: 'color: #F44336'
  };
  console[type === 'error' ? 'error' : 'log'](`%c${prefix} ${message}`, styles[type]);
}
```

**Logging must be verbose by default.** Log these events:

1. **Initialisation phase:**
   - Extension starting
   - URL pattern match/mismatch
   - Each shortcut being registered
   - Each target element found/not found during initial scan

2. **Runtime:**
   - Keyboard shortcut triggered (which one)
   - Element found/not found when shortcut fires
   - Click/focus success or failure

Example initialisation output:
```
[MyExtension] Initialising...
[MyExtension] URL matches pattern - shortcuts active
[MyExtension] Registering 3 shortcuts...
[MyExtension]   Ctrl+S → Save record
[MyExtension]   Ctrl+N → New record
[MyExtension]   Ctrl+E → Edit record
[MyExtension] Scanning for target elements...
[MyExtension]   ✓ Found "Save" button
[MyExtension]   ✓ Found "New" button
[MyExtension]   ✗ "Edit" button not found (will retry when triggered)
[MyExtension] Ready!
```

## File structure

```
extension-folder/
├── manifest.json       # Extension configuration (use template below)
├── content.js          # Main script injected into pages
├── styles.css          # Toast and tooltip styles
├── options.html        # Settings page
├── options.js          # Settings page logic
└── icons/
    ├── icon16.png      # Toolbar icon
    ├── icon48.png      # Extensions page
    └── icon128.png     # Chrome Web Store
```

## manifest.json template

Always use Manifest V3 (not V2). Use this exact structure:

```json
{
  "manifest_version": 3,
  "name": "EXTENSION_NAME",
  "version": "1.0.0",
  "description": "DESCRIPTION",
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  },
  "permissions": [
    "storage"
  ],
  "action": {
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png"
    },
    "default_title": "EXTENSION_NAME"
  },
  "options_ui": {
    "page": "options.html",
    "open_in_tab": true
  },
  "content_scripts": [
    {
      "matches": ["MATCH_PATTERNS"],
      "js": ["content.js"],
      "css": ["styles.css"],
      "run_at": "document_idle"
    }
  ]
}
```

**Match pattern examples:**
- Single site: `"https://app.salesforce.com/*"`
- All subdomains: `"https://*.salesforce.com/*"`
- Multiple: `["https://*.salesforce.com/*", "https://*.force.com/*"]`

## Platform-specific keyboard handling

Detect platform and use appropriate modifier:

```javascript
const isMac = navigator.platform.toUpperCase().indexOf('MAC') >= 0;

function matchesShortcut(event, keys) {
  // Ctrl on Mac, Alt on Windows/Linux
  const modifierPressed = isMac ? event.ctrlKey : event.altKey;
  if (!modifierPressed) return false;
  
  // Avoid Ctrl+Alt combinations
  if (isMac && event.altKey) return false;
  if (!isMac && event.ctrlKey) return false;
  
  // Only check Shift if the shortcut requires it
  if (keys.withShift && !event.shiftKey) return false;
  if (!keys.withShift && event.shiftKey) return false;
  
  return event.key.toLowerCase() === keys.key.toLowerCase();
}
```

**Shortcut configuration format:**
```javascript
{ key: 'm', withShift: false }  // Ctrl+M on Mac, Alt+M on Windows
{ key: 's', withShift: true }   // Ctrl+Shift+S on Mac, Alt+Shift+S on Windows
```

## Element finding with retries

SPAs like Salesforce load elements asynchronously. Always use retry logic with a **strict termination condition**:

```javascript
function findElement(selector, description, maxAttempts = 10, interval = 500) {
  return new Promise((resolve, reject) => {
    let attempts = 0;
    
    function tryFind() {
      attempts++;
      const element = document.querySelector(selector);
      
      if (element && isVisible(element)) {
        resolve(element);
        return;  // SUCCESS - exit
      }
      
      if (attempts >= maxAttempts) {
        reject(new Error(`Could not find "${description}" after ${maxAttempts} attempts`));
        return;  // FAILURE - exit, do NOT retry forever
      }
      
      setTimeout(tryFind, interval);
    }
    
    tryFind();
  });
}

function isVisible(element) {
  if (!element) return false;
  const rect = element.getBoundingClientRect();
  const style = window.getComputedStyle(element);
  return rect.width > 0 && rect.height > 0 && 
         style.visibility !== 'hidden' && style.display !== 'none';
}
```

**Critical**: The `maxAttempts` check must come BEFORE scheduling the next retry. Never use `while(true)` or retry indefinitely.

## Text-based element finding

When selectors are unstable, find by visible text. Handle multiple matches with index:

```javascript
function findByText(tag, text, index = 0) {
  const elements = [...document.querySelectorAll(tag)].filter(el => {
    const elText = el.textContent.trim().toLowerCase();
    return elText.includes(text.toLowerCase()) && isVisible(el);
  });
  
  if (elements.length === 0) {
    return null;
  }
  
  // If multiple matches, use the specified index
  if (elements.length > 1) {
    console.log(`Found ${elements.length} elements matching "${text}" - using index ${index}`);
  }
  
  return elements[index] || null;
}
```

Use selector first, fall back to text:
```javascript
async function findTargetElement(config) {
  // Try selector first (more reliable if stable)
  if (config.selector) {
    const el = document.querySelector(config.selector);
    if (el && isVisible(el)) return el;
  }
  
  // Fall back to text matching
  if (config.textMatch) {
    const el = findByText(config.textMatch.tag, config.textMatch.text, config.textMatch.index);
    if (el) return el;
  }
  
  throw new Error(`Could not find element: ${config.description}`);
}
```

## URL monitoring for SPAs

SPAs use pushState/replaceState instead of full page loads. Monitor URL changes:

```javascript
function setupUrlMonitoring(activePattern, onMatch, onNoMatch) {
  function checkUrl() {
    const matches = activePattern.test(window.location.href);
    if (matches) onMatch();
    else onNoMatch();
  }
  
  // Initial check
  checkUrl();
  
  // Intercept History API
  const originalPushState = history.pushState;
  const originalReplaceState = history.replaceState;
  
  history.pushState = function(...args) {
    originalPushState.apply(this, args);
    setTimeout(checkUrl, 100);
  };
  
  history.replaceState = function(...args) {
    originalReplaceState.apply(this, args);
    setTimeout(checkUrl, 100);
  };
  
  // Back/forward navigation
  window.addEventListener('popstate', () => setTimeout(checkUrl, 100));
  
  // Fallback periodic check
  setInterval(checkUrl, 2000);
}
```

## Toast notifications

**Only show toasts on failure, not success.** Successful shortcuts should be silent (the action itself is the feedback).

Key requirements for error toasts:
- Position: fixed, bottom-right
- z-index: 2147483647 (maximum, ensures visibility over any page content)
- Auto-dismiss after ~5 seconds
- Click to dismiss
- Red/error colour
- Include actionable hint: "Open console (Cmd+Option+J) for details"

## Hover tooltips

Requirements:
- Position: `fixed`, top-left corner (`top: 8px; left: 8px`)
- z-index: 2147483647
- Background: #1a1a1a (near-black)
- Text: #ffffff, small font (~12px)
- Padding: minimal (~6px 10px)
- Border-radius: 4px
- **Instant show on mouseenter** — no delay, no animation
- **Instant hide on mouseleave** — no delay, no animation

```css
.ext-tooltip {
  position: fixed;
  top: 8px;
  left: 8px;
  z-index: 2147483647;
  background: #1a1a1a;
  color: #fff;
  font-size: 12px;
  padding: 6px 10px;
  border-radius: 4px;
  pointer-events: none;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
```

Show/hide with `display: none` / `display: block` — no transitions.

### Tooltip attachment pattern

**Critical**: Elements often don't exist when the extension initialises. SPAs load content dynamically and can take several seconds. Use this robust pattern:

```javascript
// Track which elements already have tooltips attached
const tooltipAttached = new WeakSet();

function attachTooltipToElement(element, shortcutText) {
  if (tooltipAttached.has(element)) return; // Already attached
  
  element.addEventListener('mouseenter', () => showTooltip(shortcutText));
  element.addEventListener('mouseleave', () => hideTooltip());
  tooltipAttached.add(element);
  log(`  Tooltip attached for "${shortcutText}"`, 'success');
}

function tryAttachAllTooltips() {
  for (const shortcut of shortcuts) {
    const element = findElementNow(shortcut); // Non-async, immediate check
    if (element) {
      attachTooltipToElement(element, formatShortcut(shortcut));
    }
  }
}

// 1. Try immediately
tryAttachAllTooltips();

// 2. Exponential backoff: retry at 0.5s, 1s, 2s, 4s, 8s (covers ~15 seconds total)
[500, 1000, 2000, 4000, 8000].forEach(delay => {
  setTimeout(tryAttachAllTooltips, delay);
});

// 3. Watch for DOM changes and re-attach (ongoing, catches everything else)
let mutationTimeout;
const observer = new MutationObserver(() => {
  clearTimeout(mutationTimeout);
  mutationTimeout = setTimeout(tryAttachAllTooltips, 200);
});
observer.observe(document.body, { childList: true, subtree: true });

// 4. Re-attach after SPA navigation (URL changes)
// Add tryAttachAllTooltips() call inside the URL monitoring onMatch callback
```

**Key points:**
- Use `WeakSet` to track attached elements (prevents duplicate listeners, auto-cleans up)
- Exponential backoff covers slow-loading SPAs (6 attempts over ~15 seconds)
- MutationObserver catches elements added at any time after that
- Re-run on URL changes for SPA navigation
- The debounced MutationObserver callback is safe because `tryAttachAllTooltips` doesn't modify the DOM

## Safe MutationObserver pattern

MutationObservers can easily cause infinite loops if the callback modifies the DOM. Always debounce:

```javascript
let mutationTimeout;
const observer = new MutationObserver(() => {
  clearTimeout(mutationTimeout);
  mutationTimeout = setTimeout(() => {
    // Safe to modify DOM here - debounced
    attachTooltipsToNewElements();
  }, 200);
});

observer.observe(document.body, { childList: true, subtree: true });
```

**Never** modify the DOM directly inside the callback without debouncing — this triggers the observer again, creating an infinite loop that freezes the tab.

## Options page

Store shortcuts in `chrome.storage.sync` so users can customise without editing code. The options page should:
- Show current shortcut bindings
- Allow changing the key (single character input)
- Allow toggling Shift on/off per shortcut
- Allow toggling tooltips on/off
- Display platform-appropriate modifier (Ctrl on Mac, Alt on Windows)

## Icon generation

Generate simple coloured icons with a letter. Create 16px, 48px, and 128px versions as PNG files in the icons/ folder.

## Common selector strategies for CRMs

### Distinguishing buttons from form fields

**Critical**: When looking for a button, don't accidentally select a form field label with the same text.

Clickable elements (for "click" actions):
- `button`, `a`, `[role="button"]`
- Salesforce: `lightning-button`, `lightning-button-icon`, `.slds-button`
- Elements with `onclick` handlers

Form elements (for "focus" actions):
- `input`, `textarea`, `select`
- Elements with `for` attribute (labels)
- Salesforce: `lightning-input`, `lightning-textarea`

**When searching for "Email"**, these are very different:
```html
<!-- WRONG - this is a form label -->
<label>Email</label>
<input type="email" placeholder="Email">

<!-- RIGHT - this is a clickable button -->
<button>Email</button>
<a href="mailto:...">Email</a>
<lightning-button label="Email">
```

### Stable attribute strategies

When IDs are dynamic, use stable attributes:

```javascript
// By aria-label (usually stable)
'button[aria-label="Save"]'

// By data attributes
'[data-action="save"]'
'[data-automation-id="save-button"]'

// By title attribute
'button[title="Save"]'

// By position in known container
'.modal-footer button:last-child'
'.toolbar button:first-of-type'

// Salesforce Lightning specific
'lightning-button[label="New"]'
'.slds-page-header__controls button'
```
