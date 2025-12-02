# Publishing to Chrome Web Store

Guide for publishing extensions to the Chrome Web Store for easier distribution and updates.

## When to use the Chrome Web Store

**Recommended when:**

* Sharing with more than 2-3 colleagues
* You need to push updates without manual redistribution
* You want version control and rollback capability
* Professional/organisational distribution

**Stick with ZIP sharing when:**

* Personal use only
* Quick testing or prototyping
* Privacy concerns about listing

## Publishing options

### 1. Unlisted

* Anyone with the direct link can install
* Not visible in store search
* Good for sharing within organisations

### 2. Private (invite-only)

* Only specific Google accounts (by email address) can install
* Not visible in store search
* Best for restricted internal team tools

### 3. Public

* Visible in Chrome Web Store search
* Anyone can install
* Requires more thorough review

## Required assets

When the user wants to publish, generate these:

### 1. Extension description (short)

Maximum 132 characters. Example:

```
Keyboard shortcuts for Salesforce: quickly save, create, and navigate records without touching your mouse.
```

### 2. Extension description (detailed)

For the store listing page:

```
Speed up your Salesforce workflow with customisable keyboard shortcuts.

FEATURES:
• Ctrl+Shift+S to save the current record
• Ctrl+Shift+N to create a new record
• Customisable shortcuts via the options page
• Visual tooltips show shortcuts when hovering over buttons

WORKS ON:
• Salesforce Lightning Experience
• Contact, Account, Lead, and Opportunity record pages

PRIVACY:
This extension runs entirely in your browser. It does not collect, store, or transmit any data. No account required.

CUSTOMISATION:
Click the extension icon and select "Options" to change keyboard shortcuts to your preference.

SUPPORT:
Having issues? Right-click the extension icon → "Options" for troubleshooting tips.
```

### 3. Screenshots

Required: at least 1 screenshot (1280×800 or 640×400 pixels)

Tell the user:

```
You'll need at least one screenshot showing the extension in action. Here's how:

1. Go to the target website with your extension active
2. Trigger a shortcut to show the toast notification
3. Take a screenshot:
   * Mac: Cmd+Shift+4, then drag to select
   * Windows: Win+Shift+S
4. The image should be 1280×800 or 640×400 pixels

Good screenshots show:
• The toast notification confirming an action
• The tooltip appearing over a button
• The options page with shortcut settings
```

### 4. Promotional images (optional but recommended)

* Small tile: 440×280 pixels
* Large tile: 920×680 pixels
* Marquee: 1400×560 pixels

### 5. Icon

Already included in the extension, but Chrome Web Store uses the 128×128 version.

## Privacy policy

### For private/unlisted extensions

Chrome Web Store now requires a privacy policy URL for all extensions. For simple internal tools that don't collect data, the user can:

**Option A: Host a simple page**

Create a Google Doc or simple webpage with:

```
Privacy Policy for [Extension Name]

Last updated: [Date]

This browser extension does not collect, store, or transmit any personal data or 
browsing information. All functionality runs entirely within your browser.

The extension:
• Does not track your browsing activity
• Does not collect any personal information
• Does not communicate with any external servers
• Does not use cookies or local storage for tracking

For questions, contact: [email]
```

**Option B: Use a privacy policy generator**

Free services like privacypolicies.com can generate a compliant policy.

### For public extensions

More comprehensive policy required. Should cover:

* What data is accessed (even if just reading the page)
* Data storage (chrome.storage.sync stores preferences)
* Third-party sharing (none)
* Contact information

## Publishing steps

Provide these instructions when the user is ready:

### Step 1: Create a developer account

1. Go to the [Chrome Web Store Developer Dashboard](https://chrome.google.com/webstore/devconsole)
2. Sign in with a Google account
3. Pay the one-time $5 USD registration fee
4. Accept the developer agreement

### Step 2: Prepare the ZIP file

1. Make sure your extension folder contains all required files:
   * manifest.json
   * content.js
   * styles.css
   * options.html
   * options.js
   * icons/ folder with 16, 48, and 128 pixel icons

2. Select all files and create a ZIP archive
   * Mac: Select files → Right-click → Compress
   * Windows: Select files → Right-click → Send to → Compressed folder

**Important**: ZIP the files directly, not the containing folder. When you open the ZIP, manifest.json should be at the root level.

### Step 3: Upload the extension

1. In the Developer Dashboard, click "New Item"
2. Upload your ZIP file
3. Fill in the store listing:
   * Name
   * Short description (132 characters max)
   * Detailed description
   * Category (Productivity)
   * Language

### Step 4: Add visual assets

1. Upload at least one screenshot
2. Add promotional images if desired
3. The icon is pulled from your manifest

### Step 5: Set visibility

Choose your distribution option:

**Unlisted:**
1. Select "Unlisted"
2. Anyone with the direct link can install
3. Not searchable in the store

**Private:**
1. Select "Private"
2. Add Google account email addresses that can install
3. Only listed users will see the extension

**Public:**
1. Select "Public"
2. Will be reviewed more thoroughly (can take days)
3. Visible to everyone in Chrome Web Store search

### Step 6: Submit for review

1. Add privacy policy URL
2. Review all information
3. Click "Submit for Review"

Review times:

* Private/Unlisted: Usually 1-2 business days
* Public: 3-7 business days or longer

### Step 7: Share the extension

Once approved, you'll get a Chrome Web Store URL like:

```
https://chrome.google.com/webstore/detail/your-extension/abcdefghijklmnop
```

Share this URL with colleagues. They can install with one click.

## Updating the extension

When you need to push updates:

1. Update the version number in manifest.json (e.g., "1.0.0" → "1.0.1")
2. Create a new ZIP file
3. In Developer Dashboard, select your extension
4. Click "Package" → "Upload new package"
5. Upload the new ZIP
6. Submit for review

**Important**: Increment the version number or the upload will fail.

Updates are automatically pushed to all users after approval. They don't need to do anything.

## Common rejection reasons

Avoid these issues:

1. **Missing privacy policy**: Required even for simple extensions
2. **Vague descriptions**: Be specific about what the extension does
3. **Requesting unnecessary permissions**: Only request what you need
4. **Poor quality screenshots**: Blurry or irrelevant images
5. **Generic names**: Don't use "Chrome Extension" or similar

## Cost summary

* Developer registration: $5 USD (one-time)
* Hosting: Free
* Updates: Free
* No ongoing costs
