# Custom Confirmations

## Overview

By default, Turbo uses the browser's native `confirm()` dialog for confirmation prompts. While functional, the native dialog is limited in styling and user experience. Custom confirmation modals provide better UX with full control over appearance and behavior.

**What you'll learn:**
- Override Turbo's default confirmation behavior
- Create reusable confirmation modals
- Pass custom data to confirmations
- Build dynamic modals per action

## Default Turbo Confirmations

Turbo provides built-in confirmation support via the `data-turbo-confirm` attribute:
```erb
<%= button_to "Delete", post_path(@post), 
    method: :delete,
    data: { turbo_confirm: "Are you sure?" } %>
```

This triggers the browser's native `confirm()` dialog with the provided message.

**Limitations of native dialogs:**
- Cannot be styled
- Limited to text-only messages
- Poor mobile UX
- Inconsistent appearance across browsers
- No customization of button text

## Custom Confirmation Pattern

Override Turbo's confirmation method to use a custom modal instead of the browser dialog.

### Basic Implementation

**JavaScript:**
```javascript
// app/javascript/confirmations.js
Turbo.config.forms.confirm = confirmMethod

function confirmMethod(message) {
  return new Promise((resolve) => {
    const dialog = document.getElementById("confirm-dialog")
    dialog.querySelector(".message").textContent = message
    dialog.showModal()
    
    dialog.querySelector(".confirm").addEventListener("click", () => {
      dialog.close()
      resolve(true)
    }, { once: true })
    
    dialog.querySelector(".cancel").addEventListener("click", () => {
      dialog.close()
      resolve(false)
    }, { once: true })
  })
}
```

**HTML:**
```html
<!-- app/views/layouts/application.html.erb -->
<dialog id="confirm-dialog" class="confirmation-modal">
  <div class="modal-content">
    <p class="message">Are you sure?</p>
    
    <div class="modal-actions">
      <button class="cancel">Cancel</button>
      <button class="confirm">Confirm</button>
    </div>
  </div>
</dialog>
```

**Usage:**
```erb
<%= button_to "Delete Post", post_path(@post), 
    method: :delete,
    data: { turbo_confirm: "Delete this post?" } %>

<%= link_to "Remove User", user_path(@user),
    data: { turbo_method: :delete, turbo_confirm: "Remove this user?" } %>
```

**How it works:**

1. User clicks button/link with `data-turbo-confirm`
2. Turbo calls your custom `confirmMethod` instead of browser `confirm()`
3. Function returns a Promise that Turbo waits on
4. Show custom modal and set up event listeners
5. User clicks "Confirm" → resolve(true) → action proceeds
6. User clicks "Cancel" → resolve(false) → action cancelled

### The Promise Pattern

The key is returning a Promise that resolves to `true` or `false`:
```javascript
function confirmMethod(message) {
  return new Promise((resolve) => {
    // Show modal
    // Set up buttons:
    //   Confirm button: resolve(true)
    //   Cancel button: resolve(false)
  })
}
```

Turbo waits for the Promise to resolve:
- `resolve(true)` - Proceed with the action (submit form, follow link)
- `resolve(false)` - Cancel the action

## Passing Custom Data

You can pass structured data to your confirmation modal using JSON.

### JSON Data Pattern

**Ruby:**
```erb
<%= button_to "Delete Post", post_path(@post),
    method: :delete,
    data: { 
      turbo_confirm: {
        title: "Delete Post?",
        message: "This will permanently delete '#{@post.title}'",
        confirm_text: "Delete",
        cancel_text: "Keep Post"
      }.to_json
    } %>
```

**JavaScript:**
```javascript
Turbo.config.forms.confirm = confirmMethod

function confirmMethod(message) {
  return new Promise((resolve) => {
    let data
    
    // Try to parse as JSON
    try {
      data = JSON.parse(message)
    } catch {
      // Fall back to string message
      data = { message: message }
    }
    
    const dialog = document.getElementById("confirm-dialog")
    
    // Update modal content
    dialog.querySelector(".title").textContent = data.title || "Confirm Action"
    dialog.querySelector(".message").textContent = data.message || message
    dialog.querySelector(".confirm").textContent = data.confirm_text || "Confirm"
    dialog.querySelector(".cancel").textContent = data.cancel_text || "Cancel"
    
    dialog.showModal()
    
    dialog.querySelector(".confirm").addEventListener("click", () => {
      dialog.close()
      resolve(true)
    }, { once: true })
    
    dialog.querySelector(".cancel").addEventListener("click", () => {
      dialog.close()
      resolve(false)
    }, { once: true })
  })
}
```

**Enhanced HTML:**
```html
<dialog id="confirm-dialog" class="confirmation-modal">
  <div class="modal-content">
    <h3 class="title">Confirm Action</h3>
    <p class="message">Are you sure?</p>
    
    <div class="modal-actions">
      <button class="cancel">Cancel</button>
      <button class="confirm">Confirm</button>
    </div>
  </div>
</dialog>
```

### Dynamic Modal Per Action

For even more control, build modals dynamically:

**Ruby:**
```erb
<%= button_to "Delete Post", post_path(@post),
    method: :delete,
    data: { 
      turbo_confirm: {
        title: "Delete Post?",
        message: "This action cannot be undone.",
        type: "danger",
        confirm_text: "Delete Forever",
        cancel_text: "Cancel"
      }.to_json
    } %>

<%= button_to "Archive Post", archive_post_path(@post),
    method: :patch,
    data: { 
      turbo_confirm: {
        title: "Archive Post?",
        message: "You can restore this later.",
        type: "warning",
        confirm_text: "Archive",
        cancel_text: "Cancel"
      }.to_json
    } %>
```

**JavaScript with dynamic styling:**
```javascript
Turbo.config.forms.confirm = confirmMethod

function confirmMethod(message) {
  return new Promise((resolve) => {
    let data
    
    try {
      data = JSON.parse(message)
    } catch {
      data = { message: message, type: "default" }
    }
    
    const dialog = document.getElementById("confirm-dialog")
    
    // Update content
    dialog.querySelector(".title").textContent = data.title || "Confirm"
    dialog.querySelector(".message").textContent = data.message || message
    dialog.querySelector(".confirm").textContent = data.confirm_text || "Confirm"
    dialog.querySelector(".cancel").textContent = data.cancel_text || "Cancel"
    
    // Apply styling based on type
    const confirmBtn = dialog.querySelector(".confirm")
    confirmBtn.className = `confirm btn-${data.type || 'primary'}`
    
    dialog.showModal()
    
    dialog.querySelector(".confirm").addEventListener("click", () => {
      dialog.close()
      resolve(true)
    }, { once: true })
    
    dialog.querySelector(".cancel").addEventListener("click", () => {
      dialog.close()
      resolve(false)
    }, { once: true })
  })
}
```

**CSS:**
```css
.btn-danger {
  background-color: #ef4444;
  color: white;
}

.btn-warning {
  background-color: #f59e0b;
  color: white;
}

.btn-primary {
  background-color: #3b82f6;
  color: white;
}
```

## Complete Example

Here's a full working implementation with all features:

**JavaScript:**
```javascript
// app/javascript/confirmations.js
Turbo.config.forms.confirm = confirmMethod

function confirmMethod(message) {
  return new Promise((resolve) => {
    let data
    
    // Parse JSON or use string
    try {
      data = JSON.parse(message)
    } catch {
      data = { 
        title: "Confirm Action",
        message: message,
        confirm_text: "Confirm",
        cancel_text: "Cancel",
        type: "default"
      }
    }
    
    const dialog = document.getElementById("confirm-dialog")
    
    // Update modal content
    dialog.querySelector(".title").textContent = data.title
    dialog.querySelector(".message").textContent = data.message
    
    const confirmBtn = dialog.querySelector(".confirm")
    const cancelBtn = dialog.querySelector(".cancel")
    
    confirmBtn.textContent = data.confirm_text
    cancelBtn.textContent = data.cancel_text
    
    // Apply type-specific styling
    confirmBtn.className = `confirm btn btn-${data.type}`
    
    // Show modal
    dialog.showModal()
    
    // Handle confirm
    confirmBtn.addEventListener("click", () => {
      dialog.close()
      resolve(true)
    }, { once: true })
    
    // Handle cancel
    cancelBtn.addEventListener("click", () => {
      dialog.close()
      resolve(false)
    }, { once: true })
    
    // Handle escape key
    dialog.addEventListener("cancel", () => {
      resolve(false)
    }, { once: true })
  })
}
```

**HTML:**
```html
<!-- app/views/layouts/application.html.erb -->
<dialog id="confirm-dialog" class="confirmation-modal">
  <div class="modal-content">
    <h3 class="title"></h3>
    <p class="message"></p>
    
    <div class="modal-actions">
      <button class="cancel btn btn-secondary"></button>
      <button class="confirm btn btn-primary"></button>
    </div>
  </div>
</dialog>
```

**CSS:**
```css
.confirmation-modal {
  border: none;
  border-radius: 8px;
  padding: 0;
  max-width: 400px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
}

.confirmation-modal::backdrop {
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  padding: 24px;
}

.title {
  margin: 0 0 12px 0;
  font-size: 18px;
  font-weight: 600;
}

.message {
  margin: 0 0 24px 0;
  color: #6b7280;
}

.modal-actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
}

.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 6px;
  font-weight: 500;
  cursor: pointer;
}

.btn-primary {
  background-color: #3b82f6;
  color: white;
}

.btn-secondary {
  background-color: #e5e7eb;
  color: #374151;
}

.btn-danger {
  background-color: #ef4444;
  color: white;
}

.btn-warning {
  background-color: #f59e0b;
  color: white;
}
```

**Usage Examples:**
```erb
<!-- Simple string -->
<%= button_to "Delete", post_path(@post),
    method: :delete,
    data: { turbo_confirm: "Delete this post?" } %>

<!-- JSON with custom styling -->
<%= button_to "Delete Forever", post_path(@post),
    method: :delete,
    data: { 
      turbo_confirm: {
        title: "Permanently Delete Post",
        message: "This cannot be undone. Are you absolutely sure?",
        confirm_text: "Delete Forever",
        cancel_text: "Keep Post",
        type: "danger"
      }.to_json
    } %>

<!-- JSON for archive action -->
<%= button_to "Archive", archive_post_path(@post),
    method: :patch,
    data: { 
      turbo_confirm: {
        title: "Archive this post?",
        message: "You can restore it later from archives.",
        confirm_text: "Archive",
        cancel_text: "Cancel",
        type: "warning"
      }.to_json
    } %>
```

## Best Practices

### Keep Modal Focused

The confirmation modal should have one clear purpose: confirm or cancel an action.
```javascript
// ✅ Good - focused confirmation
function confirmMethod(message) {
  return new Promise((resolve) => {
    showModal(message)
    setupButtons(resolve)
  })
}

// ❌ Bad - doing too much
function confirmMethod(message) {
  return new Promise((resolve) => {
    showModal(message)
    setupButtons(resolve)
    logAnalytics()      // Don't do this
    updateUserStats()   // Don't do this
    sendNotification()  // Don't do this
  })
}
```

### Provide Clear Actions

Button text should clearly indicate what will happen:
```erb
<!-- ✅ Good - specific action -->
data: { turbo_confirm: {
  confirm_text: "Delete Post",
  cancel_text: "Keep Post"
}.to_json }

<!-- ❌ Vague - unclear outcome -->
data: { turbo_confirm: {
  confirm_text: "Yes",
  cancel_text: "No"
}.to_json }
```

### Handle Cancellation Properly

Always provide a way to cancel:
```javascript
// ✅ Good - multiple cancel paths
confirmBtn.addEventListener("click", () => {
  dialog.close()
  resolve(true)
}, { once: true })

cancelBtn.addEventListener("click", () => {
  dialog.close()
  resolve(false)
}, { once: true })

// Handle escape key and backdrop click
dialog.addEventListener("cancel", () => {
  resolve(false)
}, { once: true })
```

### Use Appropriate Visual Hierarchy

Make dangerous actions visually distinct:
```css
/* Dangerous actions - red/prominent */
.btn-danger {
  background-color: #ef4444;
  color: white;
}

/* Safe actions - subdued */
.btn-secondary {
  background-color: #e5e7eb;
  color: #374151;
}
```

For destructive actions, consider making the safe option (cancel) more prominent.

### Clean Up Event Listeners

Use `{ once: true }` to automatically remove listeners after use:
```javascript
// ✅ Good - auto cleanup
button.addEventListener("click", handler, { once: true })

// ❌ Bad - potential memory leak
button.addEventListener("click", handler)
// Listener stays attached even after modal closes
```

### Fallback for Non-JSON Messages

Always handle both string and JSON messages:
```javascript
try {
  data = JSON.parse(message)
} catch {
  // Fallback to simple string
  data = { message: message }
}
```

This ensures backward compatibility with simple string confirmations.

## Accessibility Considerations

### Focus Management

When modal opens, focus should move to it:
```javascript
dialog.showModal() // <dialog> element handles focus automatically

// For custom modals, manually manage focus:
dialog.showModal()
dialog.querySelector(".cancel").focus()
```

### Keyboard Navigation

Support keyboard interactions:
```javascript
// Escape key - handled automatically by <dialog>
dialog.addEventListener("cancel", () => {
  resolve(false)
}, { once: true })

// Tab navigation - <dialog> traps focus automatically

// Enter key on confirm button - works automatically
// Spacebar on buttons - works automatically
```

### Screen Readers

Use semantic HTML and ARIA attributes:
```html
<dialog id="confirm-dialog" 
        aria-labelledby="dialog-title"
        aria-describedby="dialog-message">
  <h3 id="dialog-title" class="title">Confirm Action</h3>
  <p id="dialog-message" class="message">Are you sure?</p>
  
  <div class="modal-actions">
    <button class="cancel" aria-label="Cancel action">Cancel</button>
    <button class="confirm" aria-label="Confirm action">Confirm</button>
  </div>
</dialog>
```

## Troubleshooting

### Confirmation Not Showing

**Problem:** Custom modal doesn't appear when clicking confirm button.

**Common Causes:**
1. `Turbo.config.forms.confirm` not set
2. Dialog element missing from page
3. JavaScript not loaded

**Solution:**
```javascript
// 1. Verify config is set
console.log(Turbo.config.forms.confirm) // Should be your function

// 2. Verify dialog exists
console.log(document.getElementById("confirm-dialog")) // Should not be null

// 3. Check browser console for JavaScript errors
```

### Promise Not Resolving

**Problem:** Action never proceeds or cancels after clicking button.

**Cause:** `resolve()` not being called in event listeners.

**Solution:**
```javascript
// ❌ Wrong - forgot to call resolve
confirmBtn.addEventListener("click", () => {
  dialog.close()
  // Missing: resolve(true)
})

// ✅ Correct - always resolve
confirmBtn.addEventListener("click", () => {
  dialog.close()
  resolve(true)
}, { once: true })
```

### Dialog Not Closing

**Problem:** Modal stays open after clicking button.

**Cause:** Not calling `dialog.close()`.

**Solution:**
```javascript
confirmBtn.addEventListener("click", () => {
  dialog.close() // Must close dialog
  resolve(true)
}, { once: true })
```

### Multiple Confirmations Stack

**Problem:** Multiple modals appear stacked when confirming quickly.

**Cause:** Not cleaning up previous modal state.

**Solution:**
```javascript
function confirmMethod(message) {
  return new Promise((resolve) => {
    const dialog = document.getElementById("confirm-dialog")
    
    // Close any existing dialog first
    if (dialog.open) {
      dialog.close()
    }
    
    // Remove old event listeners by cloning
    const newDialog = dialog.cloneNode(true)
    dialog.replaceWith(newDialog)
    
    // Now set up fresh listeners
    // ...
  })
}
```

**Better approach - use `{ once: true }`:**
```javascript
// Event listeners auto-remove after one use
confirmBtn.addEventListener("click", handler, { once: true })
```

### JSON Parsing Fails

**Problem:** Error when passing JSON data.

**Cause:** Improper JSON escaping in ERB.

**Solution:**
```erb
<!-- ❌ Wrong - quotes not escaped -->
data: { turbo_confirm: "{title: "Delete"}" }

<!-- ✅ Correct - use .to_json -->
data: { turbo_confirm: { title: "Delete" }.to_json }

<!-- ✅ Also correct - proper escaping -->
data: { turbo_confirm: '{"title":"Delete"}' }
```

### Backdrop Click Doesn't Cancel

**Problem:** Clicking outside modal doesn't cancel action.

**Cause:** Not handling `cancel` event on dialog.

**Solution:**
```javascript
// <dialog> fires 'cancel' event on escape or backdrop click
dialog.addEventListener("cancel", (event) => {
  resolve(false)
}, { once: true })

// If you want to prevent backdrop click from closing:
dialog.addEventListener("cancel", (event) => {
  event.preventDefault() // Prevents closing
})
```

### Styling Not Applied

**Problem:** Modal appears unstyled or with default browser styles.

**Cause:** CSS not loaded or selectors not matching.

**Solution:**
```css
/* Reset default dialog styles */
dialog {
  border: none;
  padding: 0;
}

/* Target the dialog element directly */
#confirm-dialog {
  /* Your styles */
}

/* Target the backdrop */
#confirm-dialog::backdrop {
  background-color: rgba(0, 0, 0, 0.5);
}
```

## Additional Resources

- [HTML Dialog Element (MDN)](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog)
- [Turbo Confirmations Documentation](https://turbo.hotwired.dev/handbook/drive#turbo-drive-confirmations)
- [Promise Documentation (MDN)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)