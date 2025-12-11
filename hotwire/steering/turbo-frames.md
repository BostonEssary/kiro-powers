# Turbo Frames

## Overview

Turbo Frames decompose pages into independent contexts that can be updated, lazy-loaded, and navigated independently. Each frame acts as a mini-page within your page, with its own navigation scope and update lifecycle.

**Key Concepts:**
- Frames scope navigation - links and forms inside a frame only update that frame
- Frames can be lazy-loaded on-demand
- Frames enable inline editing without full page reloads
- Multiple frames can exist on a single page

**When to use Frames:**
- Update a specific section without affecting the rest of the page
- Lazy-load content for better initial page performance
- Create inline editing experiences
- Implement pagination or filtering within a section
- Load modal content dynamically

## Basic Usage

### Simple Frame
```
<turbo-frame id="message">
  <h1>Hello from a frame!</h1>
  <a href="/messages/1">View message</a>
</turbo-frame>
```

When the link is clicked, Turbo fetches `/messages/1` and replaces the content **inside the frame only**.

**Server response must include a matching frame:**
```
<!-- Response from /messages/1 -->
<turbo-frame id="message">
  <h1>Message #1</h1>
  <p>This is the message content.</p>
  <a href="/messages">Back</a>
</turbo-frame>
```

### Lazy Loading

Load frame content on-demand using the `src` attribute:
```
<turbo-frame id="comments" src="/posts/1/comments" loading="lazy">
  <p>Loading comments...</p>
</turbo-frame>
```

**Attributes:**
- `loading="lazy"` - Loads when frame enters viewport (default)
- `loading="eager"` - Loads immediately on page load

The content between the opening and closing tags is displayed while loading.

## Frame Attributes

### id (Required)

Every frame must have a unique `id` that matches between request and response:
```
<!-- Page frame -->
<turbo-frame id="edit-post">...</turbo-frame>

<!-- Server response frame (must match) -->
<turbo-frame id="edit-post">...</turbo-frame>
```

### src

URL to load frame content from:
```
<turbo-frame id="stats" src="/dashboard/stats">
  Loading statistics...
</turbo-frame>
```

### loading

Controls when lazy frames load:
- `lazy` - Load when scrolled into viewport
- `eager` - Load immediately

### target

Change where the response renders:
```
<turbo-frame id="modal" target="_top">
  <!-- Form submission will replace entire page -->
</turbo-frame>
```

### data-turbo-action

Controls browser history behavior:
- `advance` - Adds history entry (default for GET)
- `replace` - Replaces current history entry (default for POST)

## Navigation

### Links Inside Frames

Links inside a frame update only that frame by default:
```
<turbo-frame id="posts">
  <h2>Recent Posts</h2>
  <a href="/posts/1">Post 1</a> <!-- Updates only this frame -->
  <a href="/posts/2">Post 2</a> <!-- Updates only this frame -->
</turbo-frame>
```

### Forms Inside Frames

Forms inside a frame submit and update only that frame:
```
<turbo-frame id="edit-post">
  <%= form_with model: @post do |f| %>
    <%= f.text_field :title %>
    <%= f.submit "Save" %>
  <% end %>
</turbo-frame>
```

After submission, the response frame replaces the current frame content.

### Breaking Out of Frames

Use `data-turbo-frame="_top"` to navigate the entire page:
```
<turbo-frame id="modal">
  <h2>Edit Post</h2>
  <%= form_with model: @post %>
  
  <!-- This link navigates the full page -->
  <a href="/posts" data-turbo-frame="_top">Cancel</a>
</turbo-frame>
```

**Common use cases:**
- Cancel buttons that close modals
- "Back to list" links after inline editing
- Navigation that should exit the frame context

### Targeting Other Frames

Update a different frame using `data-turbo-frame`:
```
<turbo-frame id="search">
  <form action="/search">
    <input type="text" name="q">
    <!-- Update results frame, not search frame -->
    <button type="submit" data-turbo-frame="results">Search</button>
  </form>
</turbo-frame>

<turbo-frame id="results">
  <!-- Search results appear here -->
</turbo-frame>
```

**Common use cases:**
- Search forms updating separate results areas
- Filters updating content sections
- Buttons triggering updates in other frames

## Common Patterns

### Inline Editing

Click to edit, save in place:
```
<!-- Show view -->
<turbo-frame id="post-<%= @post.id %>">
  <h2><%= @post.title %></h2>
  <p><%= @post.body %></p>
  <%= link_to "Edit", edit_post_path(@post) %>
</turbo-frame>

<!-- Edit view (same frame ID) -->
<turbo-frame id="post-<%= @post.id %>">
  <%= form_with model: @post do |f| %>
    <%= f.text_field :title %>
    <%= f.text_area :body %>
    <%= f.submit "Save" %>
  <% end %>
</turbo-frame>
```

After save, controller redirects back to show view, replacing the form with content.

### Modal Content Loading

Load modal content on-demand:
```
<!-- Trigger link -->
<a href="/posts/new" data-turbo-frame="modal">New Post</a>

<!-- Modal frame (initially empty) -->
<turbo-frame id="modal"></turbo-frame>

<!-- Server response -->
<turbo-frame id="modal">
  <div class="modal">
    <%= form_with model: @post, data: { turbo_frame: "_top" } %>
  </div>
</turbo-frame>
```

Form submits navigate the full page (via `_top`), closing the modal naturally.

### Pagination Within Sections

Paginate without affecting the rest of the page:
```
<turbo-frame id="posts-list">
  <% @posts.each do |post| %>
    <div><%= post.title %></div>
  <% end %>
  
  <%= paginate @posts %>
</turbo-frame>
```

Pagination links automatically update only the frame.

### Lazy-Loaded Tabs

Load tab content on-demand:
```
<div class="tabs">
  <a href="/profile/posts" data-turbo-frame="tab-content">Posts</a>
  <a href="/profile/comments" data-turbo-frame="tab-content">Comments</a>
  <a href="/profile/likes" data-turbo-frame="tab-content">Likes</a>
</div>

<turbo-frame id="tab-content" src="/profile/posts">
  Loading...
</turbo-frame>
```

Each tab link targets the same frame, replacing its content.

### Search Results

Search form updates results frame:
```
<turbo-frame id="search-form">
  <%= form_with url: search_path, method: :get, data: { turbo_frame: "search-results" } do |f| %>
    <%= f.text_field :q %>
    <%= f.submit "Search" %>
  <% end %>
</turbo-frame>

<turbo-frame id="search-results">
  <!-- Results appear here -->
</turbo-frame>
```

## Frame Events

Turbo Frames fire events during their lifecycle. Use these for customization and side effects.

### turbo:before-frame-render

Fired before a frame updates its content.

**Use cases:**
- Animate content out before new content appears
- Transfer state from old content to new content
- Cancel render conditionally

**Note:** For frame animation implementation, see `animations.md`.

### turbo:frame-render

Fired after a frame renders new content.

**Use cases:**
- Initialize JavaScript for new content
- Animate content in after render
- Update related UI elements

### turbo:frame-load

Fired when a frame completes loading content from its `src`.

**Use cases:**
- Hide loading indicators
- Track lazy-load performance
- Initialize content after lazy load

### turbo:frame-missing

Fired when server response doesn't include a matching frame ID.

**Use cases:**
- Handle missing frames gracefully
- Log errors for debugging
- Provide fallback behavior

**Example:**
```
document.addEventListener("turbo:frame-missing", (event) => {
  // Redirect to the response URL instead of throwing error
  event.preventDefault();
  event.detail.visit(event.detail.response.url);
});
```

## Server-Side Setup

### Rails Controller Response

Controllers should respond with frame content when requested by a frame:
```
class PostsController < ApplicationController
  def show
    @post = Post.find(params[:id])
  end
  
  def edit
    @post = Post.find(params[:id])
  end
  
  def update
    @post = Post.find(params[:id])
    
    if @post.update(post_params)
      redirect_to @post
    else
      render :edit, status: :unprocessable_entity
    end
  end
end
```

### Matching Frame IDs

Server response must include a frame with matching ID:
```
<!-- View: posts/show.html.erb -->
<turbo-frame id="post-<%= @post.id %>">
  <h2><%= @post.title %></h2>
  <p><%= @post.body %></p>
  <%= link_to "Edit", edit_post_path(@post) %>
</turbo-frame>

<!-- View: posts/edit.html.erb -->
<turbo-frame id="post-<%= @post.id %>">
  <%= form_with model: @post do |f| %>
    <%= f.text_field :title %>
    <%= f.text_area :body %>
    <%= f.submit "Save" %>
  <% end %>
</turbo-frame>
```

After form submission, controller redirects to `show`, which renders the matching frame.

### Handling Frame vs Full Page Requests

Rails automatically handles frame requests. No special logic needed in most cases.

**Optional: Detect frame requests:**
```
if turbo_frame_request?
  # Render frame-specific layout or partial
else
  # Render full page
end
```

## Troubleshooting

### Frame Not Updating

**Problem:** Clicking links or submitting forms inside a frame doesn't update it.

**Common Causes:**
1. **Mismatched frame IDs** - Request and response frame IDs must match exactly
2. **Missing frame in response** - Server must return a `<turbo-frame>` tag
3. **JavaScript errors** - Check browser console for errors

**Solution:**
```
# Verify frame IDs match
<!-- Request -->
<turbo-frame id="edit-post-123">

<!-- Response (must match exactly) -->
<turbo-frame id="edit-post-123">
```

### Content Missing After Frame Update

**Problem:** Frame updates but new content doesn't appear or is incomplete.

**Causes:**
- Server response missing the frame wrapper
- Frame ID mismatch
- Content outside frame tags

**Solution:** Ensure all content is wrapped in the frame:
```
<!-- ❌ Wrong - content outside frame -->
<h2>Post Title</h2>
<turbo-frame id="post">
  <p>Post body</p>
</turbo-frame>

<!-- ✅ Correct - all content inside frame -->
<turbo-frame id="post">
  <h2>Post Title</h2>
  <p>Post body</p>
</turbo-frame>
```

### Navigation Breaking Out Unexpectedly

**Problem:** Links inside frame navigate the entire page instead of updating the frame.

**Causes:**
- Link has `data-turbo-frame="_top"`
- Link points to external domain
- Response has `target="_top"` on frame

**Solution:** Remove `data-turbo-frame="_top"` or ensure it's intentional:
```
<!-- Stays in frame -->
<a href="/posts/1">View Post</a>

<!-- Navigates full page (intentional) -->
<a href="/posts" data-turbo-frame="_top">Back to All Posts</a>
```

### Multiple Frames with Same ID

**Problem:** Multiple frames with the same ID cause unexpected behavior.

**Cause:** Frame IDs must be unique per page.

**Solution:** Use unique identifiers, typically record IDs:
```
<!-- ❌ Wrong - duplicate IDs -->
<turbo-frame id="post">...</turbo-frame>
<turbo-frame id="post">...</turbo-frame>

<!-- ✅ Correct - unique IDs -->
<turbo-frame id="post-<%= post_1.id %>">...</turbo-frame>
<turbo-frame id="post-<%= post_2.id %>">...</turbo-frame>
```

### Lazy Frame Not Loading

**Problem:** Frame with `src` and `loading="lazy"` doesn't load.

**Causes:**
- Frame never enters viewport
- Server response missing or erroring
- JavaScript errors preventing load

**Solution:** Check browser network tab for failed requests, and verify frame enters viewport:
```
<turbo-frame id="comments" src="/posts/1/comments" loading="lazy">
  <p>Loading comments...</p>
</turbo-frame>
```

If frame never enters viewport, it won't load. Consider `loading="eager"` for critical content.

### Form Validation Errors Not Displaying

**Problem:** After invalid form submission, errors don't appear in frame.

**Cause:** Controller not rendering with `status: :unprocessable_entity`.

**Solution:** Always set proper status for validation errors:
```
def update
  @post = Post.find(params[:id])
  
  if @post.update(post_params)
    redirect_to @post
  else
    # ✅ Must include status for Turbo to update frame
    render :edit, status: :unprocessable_entity
  end
end
```

### Frame Loading State Not Visible

**Problem:** No visual feedback while frame loads.

**Solution:** Add loading content between frame tags:
```
<turbo-frame id="content" src="/slow-endpoint">
  <div class="loading-spinner">
    <p>Loading...</p>
  </div>
</turbo-frame>
```

This content displays until the frame loads its `src`.

## Best Practices

- **Use unique frame IDs** - Include record IDs to prevent conflicts: `post-<%= @post.id %>`
- **Match frame IDs exactly** - Request and response frame IDs must be identical
- **Provide loading states** - Add content between frame tags for better UX
- **Use lazy loading strategically** - Lazy-load non-critical content to improve initial page load
- **Break out intentionally** - Use `data-turbo-frame="_top"` for navigation that should exit frames
- **Keep frames focused** - Each frame should represent a single, independent piece of functionality
- **Test navigation flows** - Verify links and forms behave as expected within frames

## Additional Resources

- [Turbo Frames Handbook](https://turbo.hotwired.dev/handbook/frames)
- [Turbo Frames Reference](https://turbo.hotwired.dev/reference/frames)