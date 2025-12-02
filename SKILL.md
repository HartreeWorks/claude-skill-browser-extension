---
name: browser-extension
description: Create Chrome/Chromium browser extensions for users without coding experience. Use when someone wants to add keyboard shortcuts to a website or web app, automate clicking buttons or focusing inputs on web pages, or create simple browser extensions. Primary use case is adding keyboard shortcuts to web applications like Salesforce, HubSpot, and other CRMs.
---

**Author:** Peter Hartree

**Repository:** https://github.com/HartreeWorks/claude-skill-browser-extension

**Blog post:** [Shortcuts for any web app](https://wow.pjh.is/journal/shortcuts-for-any-web-app)

---

# Browser extension skill

Create Chrome/Chromium browser extensions for users without coding experience.

## References

* `references/code-templates.md` — JavaScript patterns, CSS, and manifest templates
* `references/installation.md` — User-facing installation, troubleshooting, and sharing guides
* `references/publishing.md` — Chrome Web Store publishing guide

## Information gathering

Before writing code, ensure you have enough information. Ask clarifying questions if the request is ambiguous.

### Required information

1. **Target website URL**: Get the full URL from the browser's address bar
2. **Screenshot of the page**: A screenshot showing the buttons/elements they want to add shortcuts to
3. **HTML of the relevant section**: The HTML of the area containing the buttons (not the entire page)
4. **Desired shortcuts**: What key combinations should trigger what actions?
5. **Target elements**: What buttons/inputs/links should be clicked or focused?

### Getting a screenshot

Ask the user to take a full-page screenshot:

```
Please take a screenshot of the page showing the buttons you want to add shortcuts to:

* Mac: Cmd+Shift+3 (full screen) or Cmd+Shift+4 (select area)
* Windows: Win+Shift+S (select area) or Print Screen (full screen)
```

### Getting the page URL

If the URL seems incomplete or you're unsure about the pattern, ask:

```
Please copy the complete URL from your browser's address bar while you're on the page 
where you want the shortcuts to work. I need the full URL to set up the extension correctly.
```

### URL pattern extraction

Extract the appropriate match pattern from the provided URL. Examples:

| User provides | Match pattern | Active path check |
|---------------|---------------|-------------------|
| `https://app.hubspot.com/contacts/123456/contact/789` | `https://app.hubspot.com/*` | Check for `/contact/` in path |
| `https://innovation-data-44832.lightning.force.com/lightning/r/Contact/003QH00000MICR4YAP/view` | `https://*.lightning.force.com/*` | Check for `/lightning/r/Contact/` |
| `https://mail.google.com/mail/u/0/#inbox` | `https://mail.google.com/*` | Check for `#inbox` or specific view |

The manifest.json uses the broad match pattern, but the content script should check if the current URL path matches the user's intended view before activating shortcuts.

### Getting element information

Ask the user to share the HTML of the section containing the buttons. Important: do NOT ask for the entire `<body>` element — pages like Salesforce can have millions of characters of HTML. Instead, guide them to select just the relevant container:

```
To help me find the right elements, please share the HTML of the area containing the buttons:

1. Go to the page where you want shortcuts
2. Open Developer Tools:
   * Mac: Cmd+Option+I
   * Windows/Linux: Ctrl+Shift+I
3. Activate the element picker:
   * Mac: Cmd+Shift+C
   * Windows/Linux: Ctrl+Shift+C
4. Click on an area of the page that contains all the buttons you want shortcuts for
   * For example, click on the toolbar or header section that holds the buttons
   * Try to pick a container that includes all the buttons, but not the entire page
5. In the Elements panel, right-click the highlighted element
6. Choose "Copy" → "Copy outerHTML"
7. Paste it here

Also tell me what text appears on each button you want a shortcut for.
```

## Keyboard shortcut guidelines

### Platform-specific modifiers

Use these modifiers automatically without asking the user:

| Platform | Modifier |
|----------|----------|
| Mac | Ctrl |
| Windows/Linux | Alt |

The code templates detect the platform and bind accordingly. Simply tell the user their shortcuts will be Ctrl+[key] on Mac and Alt+[key] on Windows. Do NOT ask for confirmation or permission — just implement it this way.

Only add Shift to the shortcut if the user explicitly requests it. If they ask for "Ctrl+M", implement Ctrl+M (Mac) / Alt+M (Windows), not Ctrl+Shift+M.

**Why these modifiers**: Mac apps use Cmd for system shortcuts, so Ctrl is unused. Windows uses Ctrl heavily (Ctrl+S, Ctrl+N, etc.), so Alt avoids conflicts. Don't use Alt/Option on Mac — it's for typing special characters.

### Conflict warnings

Only warn if the user explicitly requests a conflicting shortcut on Windows (e.g., Ctrl+S). On Mac, Ctrl+[key] never conflicts.

## Response workflow

### Step 1: Confirm understanding

Briefly restate which website/page the extension targets, what shortcuts will be created, and what each shortcut does.

### Step 2: Verify element selectors

**Before writing code**, explicitly verify each target element in the provided HTML.

For each shortcut the user requested, state:
1. The element you found (tag name, key attributes)
2. Why you believe this is the correct element (not a similarly-named form field)

**Common mistakes to avoid:**

| User wants | Wrong match | Right match |
|------------|-------------|-------------|
| "Email button" | `<label>Email</label>` or `<input placeholder="Email">` | `<button>Email</button>` or `<a>Email</a>` |
| "Save button" | `<span>Save</span>` inside a label | `<button>Save</button>` or `<lightning-button label="Save">` |
| "New button" | `<input value="New">` | `<button>New</button>` or `<a role="button">New</a>` |

**Verification checklist:**
- Is this a clickable element (`button`, `a`, `[role="button"]`, `lightning-button`)? 
- Or is it a form element/label (`input`, `label`, `textarea`, `select`)?
- If the user wants to "click" something, it should be a clickable element
- If the user wants to "focus" an input, then a form element is correct

**If ambiguous**, ask the user: "I found two elements with 'Email' — one is a button in the toolbar, and one is an input field label. Which one do you want the shortcut for?"

### Step 3: Generate files

Create all required files (see `references/code-templates.md`):

* `manifest.json` — Extension configuration
* `content.js` — Main script with all logic
* `styles.css` — Toast notifications and tooltip styles
* `options.html` and `options.js` — Settings page for customising shortcuts
* `icons/` — Extension icons (16, 48, 128px)

### Step 4: Review for infinite loops and memory leaks

**Before providing the extension to the user**, carefully review the code for these failure modes that can freeze the browser tab:

**MutationObserver pitfalls:**
- Does the observer callback modify the DOM? If so, it will trigger itself infinitely
- Fix: Use a flag to skip observation during your own modifications, or disconnect before modifying
- Fix: Debounce the callback with setTimeout and clearTimeout

**Retry logic pitfalls:**
- Is there a maximum attempt limit that's actually enforced?
- Does the retry loop have a proper termination condition?
- Are failed retries being logged so the user knows what happened?

**Event listener pitfalls:**
- Are listeners being added inside a loop or callback that runs multiple times?
- Fix: Check if listener is already attached (use WeakSet), or use { once: true }, or track with a flag

**setInterval pitfalls:**
- Is there any setInterval without a corresponding clearInterval?
- Could the interval callback stack up if it takes longer than the interval?
- Fix: Use setTimeout recursively instead, or ensure cleanup on page unload

**Tooltip attachment pitfalls:**
- Are tooltips attached immediately on init without waiting for elements to load?
- Fix: Use exponential backoff (attempts at 0, 0.5s, 1s, 2s, 4s, 8s) plus MutationObserver
- Fix: Track attached elements with WeakSet to prevent duplicate listeners
- Fix: Re-attach after SPA URL changes

**General checks:**
- Are there any while(true) or for(;;) loops?
- Is there any recursion without a base case?
- Do all Promises have rejection handling?

If any issues are found, fix them before proceeding.

### Step 5: Package and deliver

Save files to `/mnt/user-data/outputs/extension-name/`.

**Critical**: Create a ZIP file of the extension folder and provide the download link:

```bash
cd /mnt/user-data/outputs && zip -r extension-name.zip extension-name/
```

Provide the download link: `[Download extension-name.zip](computer:///mnt/user-data/outputs/extension-name.zip)`

### Step 6: Provide installation instructions

Use the installation guide from `references/installation.md`. This content should be shown to the user.

### Step 7: Offer follow-up help

Use the follow-up prompts from `references/installation.md`.

### Step 8: Handle troubleshooting

If the user reports issues, provide targeted guidance from `references/installation.md` or the full troubleshooting guide if the issue is unclear.

### Step 9: Handle sharing requests

If the user wants to share with colleagues, use the sharing guide from `references/installation.md`. If they want Chrome Web Store, refer to `references/publishing.md`.
