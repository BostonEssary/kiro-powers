---
name: "hotwire"
displayName: "Hotwire"
description: "Complete Hotwire guide covering Turbo Drive, Frames, and Streams with Stimulus integration. Includes fundamentals from official docs and advanced production patterns for animations, custom actions, and event handling."
keywords: ["hotwire", "turbo", "stimulus", "frames", "streams", "rails", "spa"]
author: "Boston"
---

# Hotwire

## Overview

Hotwire is an alternative approach to building modern web applications without using much JavaScript by sending HTML instead of JSON over the wire. It consists of **Turbo** (Drive, Frames, Streams) and **Stimulus**.

**What is Turbo?**
- **Turbo Drive**: Accelerates navigation by converting page loads into AJAX requests
- **Turbo Frames**: Decomposes pages into independent contexts that can be lazy-loaded and updated independently
- **Turbo Streams**: Delivers real-time page changes over WebSocket connections or in response to form submissions

**What is Stimulus?**
A modest JavaScript framework that pairs perfectly with Turbo for progressive enhancement of HTML rendered on the server.

## Available Steering Files

This power uses modular steering files to preserve context. Load only what you need based on the task at hand.

### Fundamentals
- **turbo-drive** - SPA-like navigation without writing JavaScript
- **turbo-frames** - Decompose pages into independent contexts for scoped updates
- **turbo-streams** - Deliver real-time page updates and handle form responses
- **stimulus** - JavaScript framework for progressive enhancement with Turbo

### Advanced Patterns
- **events** - Turbo event lifecycle and common event handlers for customization
- **animations** - Adding animations to Turbo Drive (View Transitions), Frames, and Stream actions
- **custom-actions** - Creating custom Turbo Stream actions with Ruby helpers
- **confirmations** - Custom confirmation modals with Turbo Drive configuration
- **troubleshooting** - Common issues, debugging techniques, and gotchas

## When to Use What

**Turbo Drive** (automatic):
- Default behavior for all links and forms
- Converts traditional navigation into SPA-like experience
- Zero configuration needed in most cases

**Turbo Frames**:
- Update a specific section of the page without full reload
- Lazy-load content on demand
- Create inline editing experiences
- Navigate within a portion of the page

**Turbo Streams**:
- Real-time updates via WebSocket (ActionCable)
- Multiple simultaneous updates to different parts of the page
- Append/prepend/update/remove operations after form submissions
- Complex UI updates that require multiple changes

**Stimulus**:
- Add interactive behavior to server-rendered HTML
- Handle DOM events and state management
- Bridge the gap when Turbo alone isn't enough
- Keep JavaScript modular and maintainable

## Getting Started

1. **New to Hotwire?** Start with the fundamentals steering files in order:
   - Read `turbo-drive` to understand the foundation
   - Then `turbo-frames` for scoped updates
   - Then `turbo-streams` for real-time updates
   - Finally `stimulus` for JavaScript enhancement

2. **Working on a specific feature?** Jump directly to the relevant steering file:
   - Adding animations (page, frame, or stream)? → `animations`
   - Custom stream behaviors? → `custom-actions`
   - Need confirmation dialogs? → `confirmations`
   - Understanding Turbo events? → `events`
   - Something not working? → `troubleshooting`

3. **Understanding events?** Read `events` to learn the Turbo event lifecycle and how to hook into page transitions, frame loads, and stream updates.

## Best Practices

- **Start with Turbo Drive** - It's enabled by default and requires no code
- **Use Frames for scoped updates** - Don't reach for Streams when Frames suffice
- **Keep custom Stream actions focused** - Each should do one thing well
- **Prefer server-rendered HTML** - Let Turbo handle navigation and updates
- **Use Stimulus sparingly** - Only when server-side rendering isn't enough
- **Follow progressive enhancement** - Ensure functionality without JavaScript when possible

## Additional Resources

- Official Hotwire documentation: https://hotwired.dev
- Turbo Handbook: https://turbo.hotwired.dev/handbook/introduction
- Stimulus Handbook: https://stimulus.hotwired.dev/handbook/introduction