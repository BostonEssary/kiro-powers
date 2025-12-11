# Custom Turbo Stream Actions

## Overview

Turbo Streams come with seven built-in actions (append, prepend, replace, update, remove, before, after), but you can create custom actions to handle specialized behaviors that don't fit the default set.

**What are custom actions?**
Custom Turbo Stream actions are JavaScript functions that extend Turbo's capabilities, allowing you to create reusable behaviors like animations, UI feedback, or third-party library integration.

**When to create custom actions:**
- Adding animations to stream operations
- Integrating third-party libraries (toast notifications, modals)
- Performing CSS class manipulations
- Creating reusable UI patterns

**Key principle:** Custom actions should do **one thing well**. Keep them small, focused, and reusable.

## The Seven Default Actions

Before creating custom actions, remember the built-in actions:
- **append** / **prepend** - Add content to lists
- **replace** / **update** - Modify existing content
- **remove** - Delete elements
- **before** / **after** - Insert at specific positions

Create custom actions only when these don't meet your needs.

## Creating Custom Actions

Custom actions require two parts:
1. **JavaScript** - The action implementation
2. **Ruby Helper** - Rails method to generate the stream tag

### JavaScript Side

Register custom actions by extending `Turbo.StreamActions`:
```javascript
// app/javascript/turbo_streams/custom_actions.js
Turbo.StreamActions.action_name = function() {
  // `this` refers to the <turbo-stream> element
  // Access attributes with this.getAttribute('attribute_name')
  // Access targets with this.targetElements
  
  this.targetElements.forEach(element => {
    // Perform action on each target
  })
}
```

**Key concepts:**
- `this` - The `<turbo-stream>` element itself
- `this.getAttribute('name')` - Access custom attributes from the stream tag
- `this.targetElements` - Array of all elements matching the target selector
- Actions run synchronously when the stream is received

### Ruby Helper Side

Create Rails helpers to generate custom stream tags:
```ruby
# app/helpers/turbo_stream_actions_helper.rb
module TurboStreamActionsHelper
  def action_name(target, param1:, param2:)
    turbo_stream_action_tag :action_name, 
                            target: target, 
                            param1: param1,
                            param2: param2
  end
end

# Make helpers available to turbo_stream
Turbo::Streams::TagBuilder.prepend(TurboStreamActionsHelper)
```

**Why prepend to TagBuilder?**

The `turbo_stream` helper in Rails is an instance of `Turbo::Streams::TagBuilder`. By prepending your module, you make your custom actions available directly on `turbo_stream`:
```erb
<!-- Without prepend - doesn't work -->
<%= action_name "target", param1: "value" %>

<!-- With prepend - works! -->
<%= turbo_stream.action_name "target", param1: "value" %>
```

Prepending adds your methods to the method lookup chain, so `turbo_stream.action_name` calls your helper just like the built-in actions.

## Example 1: Add CSS Class (Simple)

A basic custom action that adds CSS classes to elements.

**JavaScript:**
```javascript
Turbo.StreamActions.add_css_class = function() {
  const className = this.getAttribute('class_name')

  this.targetElements.forEach(element => {
    if (element) {
      element.classList.add(className)
    }
  })
}
```

**Ruby Helper:**
```ruby
module TurboStreamActionsHelper
  def add_css_class(target, class_name:)
    turbo_stream_action_tag :add_css_class, target: target, class_name: class_name
  end
end

Turbo::Streams::TagBuilder.prepend(TurboStreamActionsHelper)
```

**Usage:**
```erb
<!-- Controller: update.turbo_stream.erb -->
<%= turbo_stream.add_css_class "post-#{@post.id}", class_name: "highlight" %>
```

**Use case:** Highlight elements after update without replacing them.

## Example 2: Remove with Animation (Animation + Timing)

Remove elements with an exit animation before removal.

**JavaScript:**
```javascript
Turbo.StreamActions.remove_with_animation = function() {
  const className = this.getAttribute('class_name')

  this.targetElements.forEach(element => {
    if (element) {
      element.classList.add(className)

      // Fallback timeout in case animationend doesn't fire
      const fallbackTimeout = setTimeout(() => element.remove(), 1000)

      element.addEventListener('animationend', () => {
        clearTimeout(fallbackTimeout)
        element.remove()
      }, { once: true })
    }
  })
}
```

**Ruby Helper:**
```ruby
module TurboStreamActionsHelper
  def remove_with_animation(target, class_name:)
    turbo_stream_action_tag :remove_with_animation, target: target, class_name: class_name
  end
end

Turbo::Streams::TagBuilder.prepend(TurboStreamActionsHelper)
```

**CSS:**
```css
@keyframes fade-out-scale {
  from {
    opacity: 1;
    transform: scale(1);
  }
  to {
    opacity: 0;
    transform: scale(0.8);
  }
}

.fade-out-scale {
  animation: fade-out-scale 0.3s ease-out forwards;
}
```

**Usage:**
```erb
<!-- Controller: destroy.turbo_stream.erb -->
<%= turbo_stream.remove_with_animation dom_id(@topic), class_name: "fade-out-scale" %>
```

**Why the fallback timeout?**
The `animationend` event sometimes doesn't fire (display: none elements, interrupted animations). The fallback ensures the element is always removed, preventing stuck UI states.

**Use case:** Provide visual feedback when deleting items from lists.

## Example 3: Toast Notification (External Library)

Show toast notifications using an external library (Toastify).

**JavaScript:**
```javascript
// Assumes Toastify is loaded via CDN or npm
Turbo.StreamActions.toast = function() {
  const message = this.getAttribute('message')
  const type = this.getAttribute('type') || 'info'

  Toastify({
    text: message,
    className: type,
    close: true,
    gravity: "top",
    position: "right"
  }).showToast()
}
```

**Ruby Helper:**
```ruby
module TurboStreamActionsHelper
  def toast(message, type: :info)
    turbo_stream_action_tag :toast, message: message, type: type
  end
end

Turbo::Streams::TagBuilder.prepend(TurboStreamActionsHelper)
```

**Usage:**
```erb
<!-- Controller: create.turbo_stream.erb -->
<%= turbo_stream.append "posts", @post %>
<%= turbo_stream.toast "Post created successfully!", type: "success" %>
```

**Note about external libraries:**
Custom actions can integrate with external npm packages or CDN-loaded libraries. However, use sparingly - keep your dependencies minimal and prefer native solutions when possible.

## External Libraries

Custom Turbo Stream actions can use external JavaScript libraries to provide enhanced functionality.

**When to use external libraries:**
- Native solutions are complex or limited (notifications, charts, animations)
- The library is already used elsewhere in your app
- The functionality is clearly superior to custom implementation

**Use sparingly:**
- Each library adds bundle size and complexity
- Native browser APIs are improving rapidly
- Maintenance overhead increases with dependencies

**Example libraries that pair well with custom actions:**
- **Toastify** - Toast notifications
- **Tippy.js** - Tooltips and popovers
- **Chart.js** - Data visualization
- **Animate.css** - Pre-built CSS animations

**Best practice:** Always consider if the functionality can be achieved with vanilla JavaScript and CSS first. Add libraries only when the benefit clearly outweighs the cost.

## Bad Example: Doing Too Much

Here's an example of a custom action that violates the "one thing well" principle:

**❌ Bad - Action does too many things:**
```javascript
// DON'T DO THIS
Turbo.StreamActions.update_dashboard = function() {
  // Updates multiple unrelated parts of the page
  const stats = this.getAttribute('stats')
  const notifications = this.getAttribute('notifications')
  const messages = this.getAttribute('messages')
  
  // Update stats
  document.getElementById('stats').innerHTML = stats
  
  // Update notifications
  document.getElementById('notifications').innerHTML = notifications
  
  // Update messages
  document.getElementById('messages').innerHTML = messages
  
  // Trigger animations
  document.querySelectorAll('.stat-card').forEach(card => {
    card.classList.add('pulse')
  })
  
  // Show toast
  showToast('Dashboard updated!')
}
```

**Why this is bad:**
1. **Does too many things** - Updates multiple sections, adds animations, shows toast
2. **Not reusable** - Hardcoded IDs make it specific to one page
3. **Hard to maintain** - Changes to any part require modifying the entire action
4. **Violates separation of concerns** - Mixing data updates, UI feedback, and animations
5. **Defeats Turbo's purpose** - Should use multiple standard stream actions instead

**✅ Better approach - Use multiple focused actions:**
```erb
<!-- Controller: dashboard.turbo_stream.erb -->
<%= turbo_stream.update "stats", partial: "dashboard/stats" %>
<%= turbo_stream.update "notifications", partial: "dashboard/notifications" %>
<%= turbo_stream.update "messages", partial: "dashboard/messages" %>
<%= turbo_stream.add_css_class ".stat-card", class_name: "pulse" %>
<%= turbo_stream.toast "Dashboard updated!" %>
```

**Key takeaway:** If your custom action updates multiple unrelated elements or performs multiple unrelated operations, break it into smaller, focused actions. Each action should have a single, clear purpose.

## Best Practices

### Keep Actions Small and Focused

Each custom action should do one thing well:
```javascript
// ✅ Good - focused on one task
Turbo.StreamActions.highlight = function() {
  const className = this.getAttribute('class_name')
  this.targetElements.forEach(el => el.classList.add(className))
}

// ❌ Bad - doing multiple things
Turbo.StreamActions.highlight_and_notify = function() {
  const className = this.getAttribute('class_name')
  this.targetElements.forEach(el => el.classList.add(className))
  showToast('Element highlighted!') // Should be separate action
}
```

### Always Provide Ruby Helpers

Never write custom stream actions without Ruby helpers:
```erb
<!-- ❌ Bad - raw turbo-stream tag -->
<turbo-stream action="add_css_class" target="post" class_name="highlight">
</turbo-stream>

<!-- ✅ Good - Ruby helper -->
<%= turbo_stream.add_css_class "post", class_name: "highlight" %>
```

Ruby helpers:
- Provide consistent API
- Handle attribute escaping
- Make refactoring easier
- Self-document usage

### Handle Missing Elements Gracefully

Always check if elements exist before operating on them:
```javascript
Turbo.StreamActions.my_action = function() {
  this.targetElements.forEach(element => {
    if (element) {
      // Perform action
      element.classList.add('active')
    }
    // Silently skip if element doesn't exist
  })
}
```

### Use Descriptive Attribute Names

Make custom attributes clear and specific:
```javascript
// ✅ Good - clear attribute names
const duration = this.getAttribute('duration')
const animationClass = this.getAttribute('animation_class')

// ❌ Bad - vague names
const time = this.getAttribute('time')
const class = this.getAttribute('class')
```

### Include Fallbacks for Timing

When using events like `animationend`, always include fallback timeouts:
```javascript
Turbo.StreamActions.fade_out = function() {
  this.targetElements.forEach(element => {
    element.classList.add('fade-out')
    
    // Fallback in case animationend doesn't fire
    const fallback = setTimeout(() => element.remove(), 1000)
    
    element.addEventListener('animationend', () => {
      clearTimeout(fallback)
      element.remove()
    }, { once: true })
  })
}
```

### Namespace Your Actions

Prefix custom actions to avoid conflicts:
```javascript
// ✅ Good - namespaced
Turbo.StreamActions.app_highlight = function() { }
Turbo.StreamActions.app_dismiss = function() { }

// ❌ Risky - might conflict with future Turbo actions
Turbo.StreamActions.show = function() { }
Turbo.StreamActions.hide = function() { }
```

## Common Patterns

### Adding Classes

Simple class manipulation:
```javascript
Turbo.StreamActions.add_class = function() {
  const className = this.getAttribute('class_name')
  this.targetElements.forEach(el => el.classList.add(className))
}
```

### Removing Classes
```javascript
Turbo.StreamActions.remove_class = function() {
  const className = this.getAttribute('class_name')
  this.targetElements.forEach(el => el.classList.remove(className))
}
```

### Toggling Classes
```javascript
Turbo.StreamActions.toggle_class = function() {
  const className = this.getAttribute('class_name')
  this.targetElements.forEach(el => el.classList.toggle(className))
}
```

### Delayed Actions

Execute action after a delay:
```javascript
Turbo.StreamActions.delayed_remove = function() {
  const delay = parseInt(this.getAttribute('delay')) || 3000
  
  this.targetElements.forEach(element => {
    setTimeout(() => element.remove(), delay)
  })
}
```

### Scroll Into View

Scroll target into view:
```javascript
Turbo.StreamActions.scroll_to = function() {
  const behavior = this.getAttribute('behavior') || 'smooth'
  
  this.targetElements.forEach(element => {
    element.scrollIntoView({ behavior, block: 'center' })
  })
}
```

### Focus Element

Set focus on target:
```javascript
Turbo.StreamActions.focus = function() {
  this.targetElements.forEach(element => {
    if (element) {
      element.focus()
      
      // Optional: select text if input
      if (element.select) {
        element.select()
      }
    }
  })
}
```

## Troubleshooting

### Custom Action Not Firing

**Problem:** Custom action doesn't execute when stream is received.

**Common Causes:**
1. JavaScript not loaded or registered
2. Typo in action name
3. JavaScript error preventing registration

**Solution:**
```javascript
// 1. Verify action is registered
console.log(Turbo.StreamActions.your_action) // Should not be undefined

// 2. Check for typos
// ❌ Wrong
<%= turbo_stream.add_class ... %>
Turbo.StreamActions.addclass = function() { }

// ✅ Correct - names must match exactly
<%= turbo_stream.add_class ... %>
Turbo.StreamActions.add_class = function() { }

// 3. Check console for JavaScript errors
```

### Target Elements Not Found

**Problem:** `this.targetElements` is empty or action doesn't affect anything.

**Cause:** Target selector doesn't match any elements on the page.

**Solution:**
```javascript
Turbo.StreamActions.my_action = function() {
  console.log('Targets found:', this.targetElements.length)
  
  this.targetElements.forEach(element => {
    console.log('Processing:', element.id)
    // Your action code
  })
}
```

Check console to verify:
- Targets are being found
- Target IDs/selectors match elements on page

### Animation Not Completing

**Problem:** Animation starts but element never gets removed or class never gets cleaned up.

**Cause:** `animationend` event not firing or not being listened to correctly.

**Solution:**
```javascript
// Always include fallback timeout
Turbo.StreamActions.animated_action = function() {
  this.targetElements.forEach(element => {
    element.classList.add('animate-out')
    
    // Fallback cleanup
    const fallback = setTimeout(() => {
      element.classList.remove('animate-out')
      console.warn('Animation timeout - fallback triggered')
    }, 1000)
    
    // Primary cleanup
    element.addEventListener('animationend', () => {
      clearTimeout(fallback)
      element.classList.remove('animate-out')
    }, { once: true })
  })
}
```

### Ruby Helper Not Available

**Problem:** Calling `turbo_stream.custom_action` throws "undefined method" error.

**Cause:** Helper module not prepended to `Turbo::Streams::TagBuilder`.

**Solution:**
```ruby
# app/helpers/turbo_stream_actions_helper.rb
module TurboStreamActionsHelper
  def custom_action(target, **options)
    turbo_stream_action_tag :custom_action, target: target, **options
  end
end

# This line is REQUIRED
Turbo::Streams::TagBuilder.prepend(TurboStreamActionsHelper)
```

Without the prepend, your helpers exist but aren't available on `turbo_stream`.

### External Library Not Working

**Problem:** Custom action using external library throws "undefined" error.

**Cause:** Library not loaded or loaded after Turbo processes the stream.

**Solution:**
```javascript
Turbo.StreamActions.toast = function() {
  // Check if library is available
  if (typeof Toastify === 'undefined') {
    console.error('Toastify not loaded')
    return
  }
  
  const message = this.getAttribute('message')
  Toastify({ text: message }).showToast()
}
```

Ensure external libraries are loaded before your custom actions:
- Load from CDN in `<head>`
- Import via npm and include in bundle
- Verify load order in browser console

### Multiple Actions Conflicting

**Problem:** Multiple custom actions targeting same element cause unexpected behavior.

**Cause:** Actions modifying same properties or classes.

**Solution:**
```erb
<!-- ❌ Bad - both actions modify classes -->
<%= turbo_stream.add_class "post", class_name: "highlight" %>
<%= turbo_stream.remove_class "post", class_name: "highlight" %>

<!-- ✅ Good - combine into single operation -->
<%= turbo_stream.toggle_class "post", class_name: "highlight" %>
```

Or use different class names for different actions:
```erb
<%= turbo_stream.add_class "post", class_name: "highlight" %>
<%= turbo_stream.add_class "post", class_name: "pulse" %>
<!-- Both classes can coexist -->
```

## Additional Resources

- [Turbo Streams Reference](https://turbo.hotwired.dev/reference/streams)
- [Turbo Streams Handbook](https://turbo.hotwired.dev/handbook/streams)
- For animation-specific custom actions, see `animations.md`