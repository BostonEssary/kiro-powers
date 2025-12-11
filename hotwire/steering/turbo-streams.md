# Turbo Streams

## Overview

Turbo Streams deliver targeted page changes by sending HTML fragments wrapped in `<turbo-stream>` elements. Unlike Turbo Frames which replace entire frame contexts, Streams can perform surgical updates to multiple parts of the page simultaneously with a single response.

**Key Concepts:**
- Streams perform specific actions (append, prepend, replace, update, remove)
- Multiple stream actions can be combined in one response
- Streams target elements by ID or CSS selector
- Perfect for form responses that need to update multiple areas

**When to use Streams:**
- Update multiple parts of the page after form submission
- Add/remove items from lists dynamically
- Show flash messages or notifications
- Perform complex UI updates that require several changes
- Real-time updates (covered in future broadcasts section)

**Streams vs Frames vs Drive:**
- **Drive**: Full page navigation (automatic)
- **Frames**: Replace one scoped section (single target)
- **Streams**: Multiple targeted updates in one response

## Basic Usage

### Single Stream Action

After form submission, respond with a Turbo Stream that appends new content:
```erb
<!-- Response: create.turbo_stream.erb -->
<turbo-stream action="append" target="posts">
  <template>
    <div id="post-<%= @post.id %>">
      <h3><%= @post.title %></h3>
      <p><%= @post.body %></p>
    </div>
  </template>
</turbo-stream>
```

This appends the new post to an element with `id="posts"` on the page.

### Multiple Stream Actions

Combine several updates in one response:
```erb
<!-- Response: create.turbo_stream.erb -->
<turbo-stream action="append" target="posts">
  <template>
    <%= render @post %>
  </template>
</turbo-stream>

<turbo-stream action="update" target="post-count">
  <template>
    <%= @posts.count %> posts
  </template>
</turbo-stream>

<turbo-stream action="update" target="flash">
  <template>
    <div class="alert alert-success">Post created successfully!</div>
  </template>
</turbo-stream>
```

All three updates happen simultaneously when the response is received.

## The Seven Actions

### append

Adds content to the **end** of the target element's children.
```html
<turbo-stream action="append" target="comments">
  <template>
    <div class="comment">New comment content</div>
  </template>
</turbo-stream>
```

**Use cases:**
- Adding new items to the bottom of lists
- Appending log entries
- Adding messages to chat

### prepend

Adds content to the **beginning** of the target element's children.
```html
<turbo-stream action="prepend" target="notifications">
  <template>
    <div class="notification">You have a new message</div>
  </template>
</turbo-stream>
```

**Use cases:**
- Adding newest items to top of lists
- Prepending notifications
- Showing most recent activity first

### replace

Replaces the **entire target element** (including the element itself).
```html
<turbo-stream action="replace" target="post-123">
  <template>
    <div id="post-123">
      <h3>Updated Post Title</h3>
      <p>Updated content</p>
    </div>
  </template>
</turbo-stream>
```

**Use cases:**
- Updating entire components after edit
- Replacing cards or list items
- Swapping UI sections completely

**Important:** The replacement content must include an element with the same ID.

### update

Replaces the **content inside** the target element (keeps the target element itself).
```html
<turbo-stream action="update" target="post-body">
  <template>
    <p>This is the new body content</p>
  </template>
</turbo-stream>
```

**Use cases:**
- Updating counters or stats
- Changing text content
- Updating inner HTML without replacing wrapper

**Difference from replace:** Target element stays, only its children change.

### remove

Removes the target element from the page entirely.
```html
<turbo-stream action="remove" target="post-123">
</turbo-stream>
```

**Use cases:**
- Deleting items from lists
- Removing notifications after timeout
- Hiding UI elements

**Note:** No `<template>` needed since we're removing, not adding.

### before

Inserts content **before** the target element (as a sibling).
```html
<turbo-stream action="before" target="post-5">
  <template>
    <div id="post-4">Content inserted before post-5</div>
  </template>
</turbo-stream>
```

**Use cases:**
- Inserting items at specific positions in lists
- Adding elements relative to existing ones

**Less common** - typically append/prepend handle most list insertion needs.

### after

Inserts content **after** the target element (as a sibling).
```html
<turbo-stream action="after" target="post-5">
  <template>
    <div id="post-6">Content inserted after post-5</div>
  </template>
</turbo-stream>
```

**Use cases:**
- Inserting items at specific positions
- Adding related content next to existing elements

**Less common** - typically append/prepend handle most list insertion needs.

## Rails Integration

### Turbo Stream Views

Create `.turbo_stream.erb` views for stream responses:
```erb
# app/views/posts/create.turbo_stream.erb
<%= turbo_stream.append "posts", @post %>
<%= turbo_stream.update "flash", partial: "shared/flash" %>
```

Rails provides helper methods that generate the `<turbo-stream>` tags:
- `turbo_stream.append(target, content)`
- `turbo_stream.prepend(target, content)`
- `turbo_stream.replace(target, content)`
- `turbo_stream.update(target, content)`
- `turbo_stream.remove(target)`

### Controller Response

Respond to turbo_stream format in controllers:
```ruby
class PostsController < ApplicationController
  def create
    @post = Post.new(post_params)
    
    if @post.save
      respond_to do |format|
        format.turbo_stream
        format.html { redirect_to @post }
      end
    else
      render :new, status: :unprocessable_entity
    end
  end
  
  def destroy
    @post = Post.find(params[:id])
    @post.destroy
    
    respond_to do |format|
      format.turbo_stream { render turbo_stream: turbo_stream.remove(@post) }
      format.html { redirect_to posts_path }
    end
  end
end
```

Rails automatically looks for `create.turbo_stream.erb` when `format.turbo_stream` is called.

### Turbo Stream Actions in HTML Responses

Turbo Stream helpers can be used inside regular HTML responses (`.html.erb` files) to perform targeted updates alongside normal content rendering. This is useful when you want to update a small piece of the page while rendering everything else normally.
```erb
<!-- show.html.erb -->
<turbo-frame id="post-details">
  <%= turbo_stream.update "notification-badge" do %>
    <span class="badge"><%= current_user.unread_count %></span>
  <% end %>
  
  <h1><%= @post.title %></h1>
  <p><%= @post.body %></p>
</turbo-frame>
```

**When to use:**
- Need to update one small piece outside the main content being rendered
- Want to trigger a side effect (like updating a counter) alongside normal HTML response
- Combining turbo actions with regular HTML in the same response

**Use sparingly** - Most turbo stream responses should use dedicated `.turbo_stream.erb` files.

## Targeting Elements

### Using IDs for Single Targets

Every stream action needs a target element with a matching ID:
```html
<!-- Page HTML -->
<div id="posts">
  <!-- Posts will be appended here -->
</div>
```
```erb
<!-- Stream response -->
<%= turbo_stream.append "posts", @post %>
```

### Multiple Targets with CSS Selectors

Use the `targets` attribute with a CSS selector to update multiple elements at once:
```erb
<!-- Multiple counters with same class -->
<span class="post-count"><%= @posts.count %></span>
<span class="post-count"><%= @posts.count %></span>

<!-- Update all elements matching the selector -->
<%= turbo_stream.update_all ".post-count", @posts.count %>
```

**Common CSS selectors:**
```erb
<%= turbo_stream.remove_all ".old-records" %>
<%= turbo_stream.append_all ".comment-lists", @comment %>
<%= turbo_stream.update_all "input.invalid-field" do %>
  <span class="error">Incorrect</span>
<% end %>
```

**Rails helper methods for multiple targets:**
- `turbo_stream.append_all(selector, content)`
- `turbo_stream.prepend_all(selector, content)`
- `turbo_stream.replace_all(selector, content)`
- `turbo_stream.update_all(selector, content)`
- `turbo_stream.remove_all(selector)`

**Note:** The `_all` suffix indicates the action targets multiple elements via CSS selector instead of a single ID.

### DOM ID Helpers in Rails

Rails provides `dom_id` helper to generate consistent IDs:
```erb
<!-- Page HTML -->
<div id="<%= dom_id(@post) %>">
  <!-- Generates id="post_123" -->
</div>

<!-- Stream response -->
<%= turbo_stream.replace dom_id(@post), @post %>
```

**Common patterns:**
```ruby
dom_id(@post)                    # => "post_123"
dom_id(@post, :edit)             # => "edit_post_123"
dom_id(@post, :comments)         # => "comments_post_123"
```

Use these to maintain consistent IDs across views and streams.

## Common Patterns

### Creating Items (Append to List)

Add new items to a list after form submission:
```erb
<!-- Page: index.html.erb -->
<div id="topics">
  <%= render @topics %>
</div>

<%= form_with model: @topic, data: { turbo_frame: "_top" } %>
```
```ruby
# Controller
def create
  @topic = @meeting.topics.create(topic_params)
  respond_to do |format|
    format.turbo_stream
  end
end
```
```erb
<!-- View: create.turbo_stream.erb -->
<%= turbo_stream.append "topics", @topic %>
<%= turbo_stream.update "flash" do %>
  <div class="alert alert-success">Topic created!</div>
<% end %>
```

### Updating Items (Replace in List)

Update existing items after edit:
```erb
<!-- Page: show.html.erb -->
<div id="<%= dom_id(@topic) %>">
  <%= render @topic %>
</div>
```
```ruby
# Controller
def update
  @topic = Topic.find(params[:id])
  
  if @topic.update(topic_params)
    respond_to do |format|
      format.turbo_stream
    end
  else
    render :edit, status: :unprocessable_entity
  end
end
```
```erb
<!-- View: update.turbo_stream.erb -->
<%= turbo_stream.replace @topic %>
```

### Deleting Items (Remove from List)

Remove items from lists with confirmation:
```erb
<!-- Partial: _topic.html.erb -->
<div id="<%= dom_id(topic) %>">
  <p><%= topic.body %></p>
  <%= button_to "Delete", 
      meeting_topic_path(topic.meeting, topic),
      method: :delete,
      data: { turbo_confirm: "Are you sure?" } %>
</div>
```
```ruby
# Controller
def destroy
  @topic = Topic.find(params[:id])
  @topic.destroy
  
  respond_to do |format|
    format.turbo_stream
  end
end
```
```erb
<!-- View: destroy.turbo_stream.erb -->
<%= turbo_stream.remove @topic %>
<%= turbo_stream.update "flash" do %>
  <div class="alert alert-info">Topic deleted</div>
<% end %>
```

## Custom Actions

Turbo Streams can be extended with custom actions beyond the seven default actions. Custom actions allow you to create specialized behaviors like animations, toasts, or any custom JavaScript behavior.

For detailed information on creating and using custom Turbo Stream actions, see `custom-actions.md`.

## Troubleshooting

### Streams Not Applying

**Problem:** Turbo Stream response received but page doesn't update.

**Common Causes:**
1. **Target element doesn't exist** - Check the ID exists on the page
2. **Invalid HTML in template** - Malformed HTML breaks stream processing
3. **JavaScript errors** - Check browser console for errors
4. **Wrong content type** - Server must send `text/vnd.turbo-stream.html`

**Solution:**
```html
# Verify target exists on page
<div id="posts">...</div>
```
```erb
# Verify stream targets correct ID
<%= turbo_stream.append "posts", @post %>
```
```
# Check response headers
Content-Type: text/vnd.turbo-stream.html; charset=utf-8
```

### Target Not Found

**Problem:** Console shows "Error: The response is missing a matching element with ID..."

**Cause:** Stream targets an ID that doesn't exist on the page.

**Solution:** Ensure target element exists before the stream response:
```html
<!-- ❌ Wrong - no element with id="posts" -->
<div class="posts-container">
  <!-- posts -->
</div>

<!-- ✅ Correct - element has matching ID -->
<div id="posts" class="posts-container">
  <!-- posts -->
</div>
```

### Multiple Stream Actions Not Working

**Problem:** Only the first stream action applies, rest are ignored.

**Cause:** Syntax error in one of the stream actions breaks processing of subsequent actions.

**Solution:** Validate each stream action individually:
```erb
<!-- Check each action separately -->
<%= turbo_stream.append "posts", @post %>
<!-- Verify this works before adding more -->

<%= turbo_stream.update "counter", @posts.count %>
<!-- Then add additional actions one at a time -->
```

### Stream Response Returns Full HTML

**Problem:** Clicking a form submit button causes full page reload instead of Turbo Stream response.

**Causes:**
- Form doesn't have Turbo enabled
- Controller not responding to turbo_stream format
- Missing turbo_stream view file

**Solution:**
```erb
# 1. Verify form uses Turbo (default in Rails 7+)
<%= form_with model: @post do |f| %>
  <!-- Turbo automatically handles submission -->
<% end %>
```
```ruby
# 2. Ensure controller responds to format
def create
  @post = Post.create(post_params)
  respond_to do |format|
    format.turbo_stream  # Must include this
    format.html { redirect_to @post }
  end
end

# 3. Create matching view file
# app/views/posts/create.turbo_stream.erb
```

### Partial Rendering Errors

**Problem:** Error when rendering partials in streams: "partial not found" or similar.

**Cause:** Incorrect partial path or missing partial file.

**Solution:**
```erb
# ❌ Wrong - incorrect path
<%= turbo_stream.append "posts", partial: "post", locals: { post: @post } %>

# ✅ Correct - full path from app/views
<%= turbo_stream.append "posts", partial: "posts/post", locals: { post: @post } %>

# ✅ Or use model rendering (Rails convention)
<%= turbo_stream.append "posts", @post %>
# Rails automatically finds app/views/posts/_post.html.erb
```

### Validation Errors Not Showing

**Problem:** After invalid form submission, validation errors don't appear.

**Cause:** Not rendering the form with proper status code.

**Solution:**
```ruby
def create
  @post = Post.new(post_params)
  
  if @post.save
    respond_to do |format|
      format.turbo_stream
    end
  else
    # ✅ Must render with unprocessable_entity status
    render :new, status: :unprocessable_entity
  end
end
```

Without the proper status, Turbo won't update the page with errors.

### Content Appearing in Wrong Location

**Problem:** Stream content appears in unexpected location on page.

**Cause:** Wrong action used (append vs prepend) or wrong target ID.

**Solution:**
```erb
# Verify action matches intent
<%= turbo_stream.append "posts" %>    # Adds to end
<%= turbo_stream.prepend "posts" %>   # Adds to beginning
<%= turbo_stream.replace "posts" %>   # Replaces entire element
<%= turbo_stream.update "posts" %>    # Updates inner content

# Verify target ID
# Use browser inspector to confirm element ID on page
```

## Best Practices

- **Use `.turbo_stream.erb` views** - Keep stream logic in views, not controllers
- **Target with IDs** - Every single target must have a unique, stable ID
- **Use CSS selectors for multiple targets** - Use `_all` helpers with class selectors
- **Use `dom_id` helpers** - Maintain consistent IDs across views and streams
- **Combine multiple actions** - Use one stream response for related updates
- **Handle validation errors** - Always render with `status: :unprocessable_entity` on errors
- **Provide fallback HTML responses** - Include `format.html` for non-Turbo requests
- **Keep templates simple** - Complex logic belongs in partials, not stream templates
- **Test without JavaScript** - Ensure HTML fallbacks work

## Additional Resources

- [Turbo Streams Handbook](https://turbo.hotwired.dev/handbook/streams)
- [Turbo Streams Reference](https://turbo.hotwired.dev/reference/streams)