# Animations

## Overview

Hotwire provides three distinct contexts for adding animations to your application, each suited for different use cases:

1. **View Transitions** - Native browser API for page-to-page animations (Turbo Drive)
2. **Turbo Frame Animations** - Animate content changes within frames
3. **Turbo Stream Animations** - Animate custom stream actions

**When to use each:**
- **View Transitions**: Smooth transitions between full page navigations or specific elements across pages
- **Frame Animations**: Animate content updates within a turbo-frame
- **Stream Animations**: Add animation to custom turbo stream actions (like animated removal)

All three approaches follow progressive enhancement principles - your application works without animations, and they enhance the experience when supported.

## View Transitions (Page-to-Page)

### What Are View Transitions?

View Transitions are a native browser API that enables smooth animations between different states of your application. When integrated with Turbo Drive, they provide seamless transitions during page navigation without writing JavaScript.

**Key Concept:** Assign a `view-transition-name` to elements you want to animate during navigation. The browser automatically animates between the old and new states.

### Basic Usage

**Whole page transition:**
```html
<!-- new.html.erb -->
<div class="max-w-xl mx-auto new-meeting">
  <nav class="mb-6">
    <%= link_to meetings_path, class: "inline-flex items-center" do %>
      Back to meetings
    <% end %>
  </nav>

  <div class="card p-6">
    <header class="mb-6">
      <h1 class="text-2xl font-bold">Create a new meeting</h1>
    </header>
    <%= render "form", meeting: @meeting %>
  </div>
</div>
```
```css
.new-meeting {
  view-transition-name: new-meeting;
}

@keyframes slide-out-left {
  from {
    transform: translateX(0);
    opacity: 1;
  }
  to {
    transform: translateX(-100%);
    opacity: 0;
  }
}

@keyframes slide-in-right {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

::view-transition-old(new-meeting) {
  animation: slide-out-left 0.3s ease-in-out forwards;
}

::view-transition-new(new-meeting) {
  animation: slide-in-right 0.3s ease-in-out forwards;
}
```

**How it works:**
1. User clicks a link to navigate
2. Browser captures the current state (old)
3. Turbo Drive loads the new page
4. Browser captures the new state (new)
5. Browser animates between old and new using your CSS

### Dynamic View Transition Names

Use dynamic IDs for animating individual items in lists:
```erb
<li class="rounded-lg p-3 bg-theme surface-card" 
    style="view-transition-name: topic-<%= card.id %>;">
  <div class="flex justify-between items-start gap-2">
    <p class="flex-1 text-theme"><%= card.body %></p>

    <%= button_to meeting_topic_path(card.meeting, card),
        method: :delete,
        data: { turbo_confirm: "Delete this topic?" },
        class: "btn-danger" do %>
      <%= render "shared/icons/x" %>
    <% end %>
  </div>

  <div class="mt-2 flex items-center justify-between">
    <span class="text-xs text-theme-muted">
      <%= time_ago_in_words(card.created_at) %> ago
    </span>

    <div class="flex items-center gap-2">
      <%= link_to edit_meeting_topic_path(card.meeting, card),
          class: "text-xs font-medium" do %>
        Edit
      <% end %>
    </div>
  </div>
</li>
```

**Why dynamic names matter:**
- Each card gets a unique `view-transition-name` (e.g., `topic-123`)
- Browser can track individual cards across navigation
- Smooth animations when cards move, update, or are removed
- Works automatically with Turbo Drive navigation

### CSS Pseudo-Elements

View Transitions use special pseudo-elements to target old and new states:
```css
/* Target the old state (before navigation) */
::view-transition-old(identifier) {
  animation: fade-out 0.3s ease-out;
}

/* Target the new state (after navigation) */
::view-transition-new(identifier) {
  animation: fade-in 0.3s ease-in;
}

/* Target both old and new */
::view-transition-group(identifier) {
  animation: scale 0.3s ease-in-out;
}
```

**Identifier** is the value you set in `view-transition-name`.

### Progressive Enhancement

View Transitions work as progressive enhancement:
- Modern browsers: Smooth animations during navigation
- Older browsers: Standard Turbo Drive navigation (still fast, no animation)
- No JavaScript required - works automatically with Turbo Drive

**Browser Support:**
- Chrome 111+
- Edge 111+
- Safari 18+
- Firefox: In development

The feature degrades gracefully - navigation still works, just without animations.

## Turbo Frame Animations

### What Are Frame Animations?

Turbo Frame animations add smooth transitions when frame content updates. Unlike View Transitions (which are automatic), frame animations require JavaScript to coordinate exit and enter animations.

**Key Events:**
- `turbo:before-frame-render` - Fires before new content replaces old content
- `turbo:frame-render` - Fires after new content is rendered

### Stimulus Controller Pattern

Create a reusable Stimulus controller to handle frame animations:
```javascript
// app/javascript/controllers/frame_animation_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  async beforeRender(event) {
    event.preventDefault()
    
    const frame = event.target
    const newFrame = event.detail.newFrame
    const exitClass = frame.dataset.exitAnimation
    const enterClass = frame.dataset.enterAnimation
    
    // Exit animation
    if (exitClass) {
      const children = Array.from(frame.children)
      children.forEach(child => child.classList.add(exitClass))
      
      await Promise.all(
        children.map(child => 
          new Promise(resolve => {
            child.addEventListener('animationend', resolve, { once: true })
          })
        )
      )
      
      children.forEach(child => child.classList.remove(exitClass))
    }
    
    // Add enter class to new content
    if (enterClass) {
      const newChildren = Array.from(newFrame.children)
      newChildren.forEach(child => child.classList.add(enterClass))
    }
    
    event.detail.resume()
  }
  
  afterRender(event) {
    const frame = event.target
    const enterClass = frame.dataset.enterAnimation
    
    if (!enterClass) return
    
    const children = Array.from(frame.children)
    children.forEach(child => {
      child.addEventListener('animationend', () => {
        child.classList.remove(enterClass)
      }, { once: true })
    })
  }
}
```

### Using the Controller
```html
<turbo-frame id="meeting-topics"
             src="/meetings/1/topics"
             data-controller="frame-animation"
             data-action="turbo:before-frame-render->frame-animation#beforeRender
                          turbo:frame-render->frame-animation#afterRender"
             data-exit-animation="fade-out"
             data-enter-animation="fade-in">
  <p>Loading topics...</p>
</turbo-frame>
```

**CSS animations:**
```css
@keyframes fade-out {
  from { opacity: 1; }
  to { opacity: 0; }
}

@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fade-out {
  animation: fade-out 0.3s ease-out forwards;
}

.fade-in {
  animation: fade-in 0.3s ease-in forwards;
}
```

### How It Works

1. **User triggers frame update** (clicks link, submits form)
2. **`turbo:before-frame-render` fires** - Controller prevents default and adds exit animation
3. **Exit animation completes** - Controller waits for `animationend` events
4. **Content switches** - Controller calls `event.detail.resume()` to allow frame update
5. **New content renders** - New content already has enter animation class
6. **`turbo:frame-render` fires** - Controller cleans up enter animation after completion

### Animation Configuration

Configure animations via data attributes:
```html
<!-- Fade animation -->
<turbo-frame data-exit-animation="fade-out" 
             data-enter-animation="fade-in">
</turbo-frame>

<!-- Slide animation -->
<turbo-frame data-exit-animation="slide-left" 
             data-enter-animation="slide-right">
</turbo-frame>

<!-- Scale animation -->
<turbo-frame data-exit-animation="scale-down" 
             data-enter-animation="scale-up">
</turbo-frame>

<!-- Only exit animation -->
<turbo-frame data-exit-animation="fade-out">
</turbo-frame>

<!-- Only enter animation -->
<turbo-frame data-enter-animation="fade-in">
</turbo-frame>
```

### Common Animation Patterns

**Fade:**
```css
@keyframes fade-out {
  from { opacity: 1; }
  to { opacity: 0; }
}

@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

**Slide:**
```css
@keyframes slide-left {
  from { transform: translateX(0); }
  to { transform: translateX(-100%); }
}

@keyframes slide-right {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}
```

**Scale:**
```css
@keyframes scale-down {
  from { transform: scale(1); opacity: 1; }
  to { transform: scale(0.8); opacity: 0; }
}

@keyframes scale-up {
  from { transform: scale(0.8); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}
```

## Turbo Stream Animations

### What Are Stream Animations?

Turbo Stream animations are implemented as custom turbo stream actions. Instead of immediately adding/removing/updating content, animated actions perform the operation with visual feedback.

**Common use case:** Animating item removal from a list

### Example: Remove with Animation

This is a brief example - see `custom-actions.md` for complete implementation details.

**Custom action:**
```javascript
Turbo.StreamActions.remove_with_animation = function() {
  const className = this.getAttribute('class_name')

  this.targetElements.forEach(element => {
    if (element) {
      element.classList.add(className)

      const fallbackTimeout = setTimeout(() => element.remove(), 1000)

      element.addEventListener('animationend', () => {
        clearTimeout(fallbackTimeout)
        element.remove()
      }, { once: true })
    }
  })
}
```

**Rails helper:**
```ruby
module TurboStreamActionsHelper
  def remove_with_animation(target, class_name:)
    turbo_stream_action_tag :remove_with_animation, target: target, class_name: class_name
  end
end

Turbo::Streams::TagBuilder.prepend(TurboStreamActionsHelper)
```

**Usage:**
```erb
<!-- Controller: destroy.turbo_stream.erb -->
<%= turbo_stream.remove_with_animation dom_id(@topic), class_name: "fade-out-scale" %>
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

### When to Use Stream Animations

Use custom stream actions with animations when:
- Removing items from lists (visual feedback before removal)
- Adding items with entrance animations
- Updating items with attention-grabbing effects
- Any stream action that benefits from visual feedback

For complete documentation on creating custom stream actions with animations, see `custom-actions.md`.

## Choosing the Right Approach

### Decision Matrix

| Use Case | Approach | Why |
|----------|----------|-----|
| Page navigation animations | View Transitions | Native browser support, zero JavaScript |
| Specific elements across pages | View Transitions | Track elements by `view-transition-name` |
| Frame content updates | Frame Animations | Smooth transitions within frame scope |
| Animated item removal | Stream Animations | Visual feedback during CRUD operations |
| Adding items with animation | Stream Animations | Custom entrance effects |
| Form submission feedback | Stream Animations | Provide visual confirmation |

### Can You Combine Them?

Yes! Each animation approach works independently:
```html
<!-- Page uses View Transitions -->
<div style="view-transition-name: page-content">
  
  <!-- Frame uses Frame Animations -->
  <turbo-frame id="topics"
               data-controller="frame-animation"
               data-exit-animation="fade-out"
               data-enter-animation="fade-in">
    
    <!-- Delete uses Stream Animation -->
    <%= button_to "Delete", topic_path(@topic), method: :delete %>
    
  </turbo-frame>
</div>
```

Each layer adds animations at different interaction points without conflicts.

## Best Practices

### Keep Animations Fast

**Recommended durations:**
- Page transitions: 200-300ms
- Frame updates: 200-400ms
- Item removal: 300-500ms

Animations longer than 500ms feel sluggish.

### Use Easing Functions

Make animations feel natural with proper easing:
```css
/* Ease out - decelerating */
animation: fade-out 0.3s ease-out;

/* Ease in - accelerating */
animation: fade-in 0.3s ease-in;

/* Ease in-out - smooth start and end */
animation: slide 0.3s ease-in-out;
```

### Provide Fallbacks

Always ensure core functionality works without animations:
```javascript
// Frame animations
if (exitClass) {
  // Animate...
} else {
  // Skip directly to content swap
  event.detail.resume()
}
```

### Avoid Animating Too Much

Don't animate everything - it becomes distracting:
```html
<!-- ❌ Bad - every frame animated -->
<turbo-frame data-exit-animation="..." data-enter-animation="...">
<turbo-frame data-exit-animation="..." data-enter-animation="...">
<turbo-frame data-exit-animation="..." data-enter-animation="...">

<!-- ✅ Good - animate important interactions -->
<turbo-frame id="main-content" data-exit-animation="..." data-enter-animation="...">
  <turbo-frame id="sidebar"><!-- No animation --></turbo-frame>
  <turbo-frame id="footer"><!-- No animation --></turbo-frame>
</turbo-frame>
```

### Test Performance

Animations can impact performance on:
- Low-end devices
- Complex layouts with many elements
- Large lists with simultaneous animations

Test on real devices and consider:
- Reducing animation complexity for large lists
- Using `will-change` CSS property sparingly
- Disabling animations based on user preference (`prefers-reduced-motion`)

### Respect User Preferences

Honor the user's motion preferences:
```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

This disables animations for users who have requested reduced motion in their system settings.

## Troubleshooting

### View Transitions Not Working

**Problem:** View Transitions don't animate during navigation.

**Common Causes:**
1. Browser doesn't support View Transitions
2. `view-transition-name` not set
3. Name conflicts (multiple elements with same name on one page)

**Solution:**
```css
/* Verify unique names */
.card-1 { view-transition-name: card-1; }
.card-2 { view-transition-name: card-2; }

/* NOT this (conflict): */
.card { view-transition-name: card; } /* Multiple elements will conflict */
```

### Frame Animations Skip or Jank

**Problem:** Frame animations don't play smoothly or skip entirely.

**Common Causes:**
1. `event.detail.resume()` called too early
2. Animation class not removed after animation
3. Multiple animations conflicting

**Solution:**
```javascript
// Wait for ALL children to finish animating
await Promise.all(
  children.map(child => 
    new Promise(resolve => {
      child.addEventListener('animationend', resolve, { once: true })
    })
  )
)

// THEN resume
event.detail.resume()
```

### Stream Animation Doesn't Remove Element

**Problem:** Element animates but never gets removed from DOM.

**Cause:** Not listening for `animationend` event or missing fallback timeout.

**Solution:**
```javascript
// Always include both animationend listener AND fallback
element.classList.add(className)

// Fallback in case animation doesn't fire
const fallbackTimeout = setTimeout(() => element.remove(), 1000)

// Primary removal on animation end
element.addEventListener('animationend', () => {
  clearTimeout(fallbackTimeout)
  element.remove()
}, { once: true })
```

### Animations Cause Layout Shift

**Problem:** Content jumps during animation.

**Cause:** Element dimensions change during animation without proper containment.

**Solution:**
```css
/* Set explicit dimensions on animated container */
.animated-frame {
  min-height: 200px; /* Prevents collapse during exit */
  position: relative;
}

/* Or use aspect ratio */
.animated-card {
  aspect-ratio: 16 / 9;
}
```

### Multiple Frame Updates Conflict

**Problem:** Rapidly updating frames causes animation conflicts.

**Cause:** New update starts before previous animation completes.

**Solution:**
```javascript
export default class extends Controller {
  async beforeRender(event) {
    // Cancel if already animating
    if (this.isAnimating) {
      event.detail.resume()
      return
    }
    
    this.isAnimating = true
    event.preventDefault()
    
    // ... perform animation ...
    
    event.detail.resume()
    this.isAnimating = false
  }
}
```

## Additional Resources

- [View Transitions API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API)
- [CSS Animations Guide (MDN)](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations)
- [Turbo Events Reference](https://turbo.hotwired.dev/reference/events)
- For custom stream actions with animations, see `custom-actions.md`