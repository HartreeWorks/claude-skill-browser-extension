# Installation instructions

User-facing content to provide after generating an extension.

## Installation guide

Provide this after the extension is ready:

```
## How to install your extension

### Step 1: Download and extract

Click the download link above to get your extension. Extract the ZIP file somewhere you'll remember (e.g., your Documents folder).

### Step 2: Open Chrome's extension page

1. Open Chrome
2. Type `chrome://extensions` in the address bar and press Enter

### Step 3: Enable Developer Mode

Find the "Developer mode" toggle in the top-right corner. Turn it **ON**.

### Step 4: Load the extension

1. Click the "Load unpacked" button (appears after enabling Developer mode)
2. Navigate to your extracted extension folder
3. Select the folder containing `manifest.json`
4. Click "Select Folder"

You should see your extension appear in the list with its icon.

### Step 5: Verify it's working

1. Go to the website your extension targets
2. Open the browser console:
   * **Mac**: Cmd+Option+J
   * **Windows/Linux**: Ctrl+Shift+J
3. Look for messages starting with `[YourExtensionName]` - you should see "Ready!" 
4. Hover over a button with a shortcut - you should see a tooltip at the top-left
5. Try one of your shortcuts!
```

## Follow-up prompts

After installation instructions, offer these:

```
Your extension is ready! Let me know if you need help:

* **Not working?** Tell me:
  - What do you see in the console? (Cmd+Option+J on Mac, Ctrl+Shift+J on Windows)
  - What happens when you press the shortcut? (Nothing? Wrong action? Error message?)

* **Want to share with colleagues?** I can explain how to distribute it — either as a simple ZIP file, or via Chrome Web Store where you can push updates automatically.

* **Want to add more shortcuts?** Or explore other ideas to reduce friction in your workflow?
```

## Troubleshooting responses

### Diagnosing from user reports

If the user describes their issue, provide targeted guidance:

**"Nothing happens when I press the shortcut"**
- Check console for messages
- Verify URL matches the pattern
- Confirm they're not in an input field

**"I see an error in the console"**
- Ask them to copy/paste the exact error
- Common: "Could not find element" → selector issue

**"The wrong button gets clicked"**
- Multiple elements matching → need to specify index
- Selector too broad → need more specific selector

**"It worked before but stopped"**
- Website updated their HTML
- Need to re-inspect and update selector

### Full troubleshooting guide

If the issue isn't clear, provide the complete guide:

```
## Troubleshooting

### Extension not appearing in chrome://extensions

* Make sure you selected the folder containing `manifest.json`, not a parent folder
* Check for red error text on the extensions page - click "Errors" to see details

### Shortcuts not working

1. Open the console:
   * **Mac**: Cmd+Option+J
   * **Windows/Linux**: Ctrl+Shift+J
2. Look for messages from your extension (they start with `[ExtensionName]`)
3. Common issues:

| Console message | Meaning | Fix |
|-----------------|---------|-----|
| No messages at all | Extension not running on this page | Check the URL matches the pattern in manifest.json |
| "URL doesn't match active pattern" | You're on a different page type | Navigate to the correct page/view |
| "Could not find element" | Selector is wrong or element loads late | Increase retry attempts or update selector |
| "Found X elements matching text" | Multiple matches | Need to specify which one (by position) |

### It was working but stopped

Websites sometimes update their HTML structure. When this happens:

1. Open Developer Tools: Cmd+Option+I (Mac) or Ctrl+Shift+I (Windows)
2. Click the Elements tab
3. Use the element picker (top-left arrow icon) to click on the button
4. Right-click the highlighted element → Copy → Copy selector
5. Share the new selector with me and I'll update the extension

### Refreshing after code changes

After editing extension files:

1. Go to `chrome://extensions`
2. Find your extension
3. Click the refresh icon (circular arrow)
4. Reload the target webpage
```

## Sharing with colleagues

When the user asks about sharing:

```
There are two ways to share your extension:

### Simple: Share the ZIP file

1. Send the ZIP file to your colleagues (email, Slack, etc.)
2. They follow the same installation steps
3. Each person loads it as an "unpacked" extension

**Limitation**: When you update the extension, everyone needs to download and reinstall manually.

### Better for teams: Chrome Web Store

Publish to the Chrome Web Store for easier management:

* Updates push automatically — colleagues get new versions without reinstalling
* Can be made available to anyone with a link, or restricted to specific email addresses
* One-time $5 registration fee, then free forever

Would you like me to walk you through publishing to the Chrome Web Store?
```
