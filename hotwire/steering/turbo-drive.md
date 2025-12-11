# Turbo Drive

## Overview

Turbo Drive is the foundation of Hotwire that automatically converts your application into a single-page application (SPA) without writing any JavaScript. It intercepts link clicks and form submissions, fetching new pages via AJAX and replacing the `<body>` while keeping the `<head>` intact.

**Benefits:**
- Faster perceived performance (no full page reloads)
- Maintains scroll position on back/forward navigation
- Preserves browser history
- Works automatically with zero configuration

**How it works:**
1. User clicks a link or submits a form
2. Turbo Drive intercepts the event
3. Makes an AJAX request to the server
4. Replaces the `<body>` with the new content
5. Updates the browser's address bar and history

## Basic Usage

Turbo Drive is **enabled by default** when you include Turbo in your application. It works automatically for:
- Standard links (`<a href="...">`)
- Form submissions (`<form>` with GET or POST)

**No configuration needed** - just build your Rails app normally with server-rendered views.

### When Turbo Drive Activates

Turbo Drive intercepts:
- Links to the same domain
- Standard HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Forms with standard submission

### When Turbo Drive Does NOT Activate

Turbo Drive skips:
- Links with `target="_blank"`
- Links to different domains
- Links with `data-turbo="false"`
- File downloads (detected by Content-Disposition header)
- Non-HTML responses

## Configuration

### Disabling Turbo Drive

**Globally (not recommended):**
```javascript
import { Turbo } from "@hotwired/turbo-rails"
Turbo.session.drive = false
```

**Per-element (recommended):**
```html
<!-- Disable for specific link -->
<a href="/legacy-page" data-turbo="false">Legacy Page</a>

<!-- Disable for entire section -->
<div data-turbo="false">
  <a href="/page1">Page 1</a>
  <a href="/page2">Page 2</a>
</div>
```

**Use cases for disabling:**
- Legacy pages that rely on full page loads
- Pages with incompatible JavaScript libraries
- File downloads or external redirects

### Permanent Elements

Elements that should persist across page navigations:
```html
<!-- Navigation bar that stays during page transitions -->
<nav id="main-nav" data-turbo-permanent>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>

<!-- Audio player that continues playing -->
<audio id="background-music" data-turbo-permanent controls>
  <source src="music.mp3" type="audio/mpeg">
</audio>
```

**Requirements:**
- Must have an `id` attribute
- Must exist on both the current and next page
- Will not be replaced during navigation

### Cache Control

**Disable caching for specific pages:**
```html
<meta name="turbo-cache-control" content="no-cache">
```

**Disable caching for specific elements:**
```html
<div data-turbo-cache="false">
  <p>User balance: $<%= current_user.balance %></p>
  <p>Last updated: <%= Time.current %></p>
</div>
```

**Use cases:**
- Pages with user-specific data
- Time-sensitive information
- Forms that shouldn't be restored from cache

### Prefetching Links

**Enable prefetching on hover (experimental):**
```html
<a href="/products" data-turbo-prefetch>Products</a>
```

Turbo will prefetch the page when the user hovers over the link, making the navigation feel instant.

## Common Patterns

### Progress Bar

Turbo Drive shows a default progress bar during navigation. Customize it with CSS:
```css
.turbo-progress-bar {
  height: 3px;
  background-color: #0076ff;
}
```

### Morphing (Selective Updates)

For pages where you want to update specific elements instead of replacing the entire body:
```html
<head>
  <meta name="turbo-refresh-method" content="morph">
  <meta name="turbo-refresh-scroll" content="preserve">
</head>
```

This is useful for:
- Refreshing data without disrupting UI state
- Maintaining scroll position during updates
- Preserving form input values

### Redirects After Form Submission

Standard Rails redirects work automatically with Turbo Drive:
```ruby
def create
  @post = Post.create(post_params)
  redirect_to @post # Turbo Drive handles this
end
```

The redirect happens via AJAX with no full page reload.

## Events

Turbo Drive fires events at different stages of navigation. Use these to customize behavior or track analytics.

### turbo:click
Fired when a link is clicked (before navigation starts).

**Use cases:**
- Cancel navigation conditionally
- Track link clicks for analytics

### turbo:before-visit
Fired before Turbo Drive visits a new page.

**Use cases:**
- Cancel navigation based on conditions
- Show custom loading UI
- Save form state before leaving

### turbo:visit
Fired when Turbo Drive starts visiting a page.

**Use cases:**
- Track page views for analytics
- Show loading indicators

### turbo:before-render
Fired before Turbo Drive renders the new page.

**Use cases:**
- Modify the new content before it's displayed
- Transfer state from old page to new page

### turbo:render
Fired after Turbo Drive renders the new page.

**Use cases:**
- Initialize JavaScript for new content
- Trigger animations
- Update UI elements

### turbo:load
Fired once after initial page load and after every Turbo Drive navigation.

**Use cases:**
- Initialize JavaScript libraries
- Set up event listeners
- Re-run setup code after navigation

**Example:**
```javascript
document.addEventListener("turbo:load", () => {
  // This runs after every page navigation
  console.log("Page loaded via Turbo Drive");
  initializeTooltips();
});
```

## Integration with Other Features

### View Transitions
Turbo Drive works seamlessly with the native View Transitions API for smooth page-to-page animations. See `animations.md` for implementation details and examples.

### Custom Confirmations
Override the default browser confirmation dialogs with custom modals. See `confirmations.md` for the full implementation pattern.

### Form Handling
Turbo Drive automatically handles form submissions via AJAX. For more control over form responses, see `turbo-streams.md` for real-time updates after submission.

## Troubleshooting

### JavaScript Not Re-initializing

**Problem:** JavaScript only runs on initial page load, not after Turbo navigation.

**Solution:** Use `turbo:load` instead of `DOMContentLoaded`:
```javascript
// ❌ Wrong - only fires on initial load
document.addEventListener("DOMContentLoaded", () => {
  initializeApp();
});

// ✅ Correct - fires after every navigation
document.addEventListener("turbo:load", () => {
  initializeApp();
});
```

### CSS/JS Not Loading After Navigation

**Problem:** Stylesheets or scripts in `<head>` aren't updating.

**Cause:** Turbo Drive only replaces `<body>` content, not `<head>`.

**Solution:** 
- Move page-specific scripts to the body
- Or use `data-turbo-track="reload"` to force full reload when assets change:
```html
<link rel="stylesheet" href="/application.css" data-turbo-track="reload">
```

### Forms Not Submitting with Turbo

**Problem:** Form submits but nothing happens.

**Solution:** Ensure your controller responds with HTML or uses Turbo Streams:
```ruby
def create
  @post = Post.create(post_params)
  
  respond_to do |format|
    format.html { redirect_to @post }
    format.turbo_stream # If using Turbo Streams
  end
end
```

### Cache Issues

**Problem:** Seeing stale data after navigation.

**Solution:** Disable cache for dynamic pages:
```html
<meta name="turbo-cache-control" content="no-cache">
```

Or clear cache programmatically:
```javascript
Turbo.cache.clear();
```

### Back Button Shows Old Content

**Problem:** Browser back button doesn't show updated content.

**Cause:** Turbo serves cached snapshots on restore visits.

**Solution:** Use `turbo:before-cache` event to prepare content for caching:
```javascript
document.addEventListener("turbo:before-cache", () => {
  // Clean up or reset elements before caching
  document.querySelectorAll(".dynamic-content").forEach(el => {
    el.textContent = "";
  });
});
```

## Best Practices

- **Start with defaults** - Turbo Drive works best when you let it handle navigation automatically
- **Use `data-turbo="false"` sparingly** - Only disable when absolutely necessary
- **Initialize JS on `turbo:load`** - Don't rely on `DOMContentLoaded` for setup code
- **Handle cache intelligently** - Use `no-cache` for user-specific or time-sensitive pages
- **Test back button behavior** - Ensure cached pages display correctly
- **Progressive enhancement** - Ensure your app works without JavaScript (forms submit, links navigate)

## Additional Resources

- [Turbo Drive Handbook](https://turbo.hotwired.dev/handbook/drive)
- [Turbo Drive Reference](https://turbo.hotwired.dev/reference/drive)