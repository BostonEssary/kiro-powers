# Stimulus

## Overview

Stimulus is a modest JavaScript framework designed for progressive enhancement of server-rendered HTML. It pairs perfectly with Turbo by providing the JavaScript sprinkles needed when HTML alone isn't enough.

**Key Philosophy:**
- Start with HTML from the server
- Add JavaScript behavior where needed
- Keep controllers small and focused
- Work with the DOM, don't fight it

**Core Concepts:**
- **Controllers**: JavaScript classes that add behavior to HTML elements
- **Targets**: Important elements within a controller's scope
- **Actions**: Methods that respond to DOM events
- **Values**: Typed data attributes that controllers can read and observe
- **Classes**: CSS class names that controllers can reference

**When to use Stimulus:**
- Add interactivity to server-rendered HTML
- Handle DOM events (clicks, inputs, changes)
- Manage UI state (toggles, modals, dropdowns)
- Enhance forms (validation, dynamic fields)
- Bridge gaps when Turbo alone isn't enough

## Core Concepts

### Controllers

Controllers are JavaScript classes that bring HTML to life. They connect to elements via `data-controller` attributes.

**Creating a controller:**
```javascript
// app/javascript/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    console.log("Hello, Stimulus!")
  }
}
```

**Connecting to HTML:**
```html
<div data-controller="hello">
  <!-- Controller is connected to this element and its children -->
</div>
```

### Targets

Targets are important elements within a controller that you need to reference. Define them in the controller and access them via `data-{controller}-target` attributes.

**Defining targets:**
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["name", "output"]
  
  greet() {
    this.outputTarget.textContent = `Hello, ${this.nameTarget.value}!`
  }
}
```

**Using targets in HTML:**
```html
<div data-controller="hello">
  <input data-hello-target="name" type="text">
  <button data-action="click->hello#greet">Greet</button>
  <p data-hello-target="output"></p>
</div>
```

**Target accessors:**
- `this.nameTarget` - Returns first matching target (throws error if missing)
- `this.nameTargets` - Returns array of all matching targets
- `this.hasNameTarget` - Returns boolean if target exists

### Actions

Actions connect DOM events to controller methods using `data-action` attributes.

**Basic syntax:**
```
data-action="{event}->{controller}#{method}"
```

**Examples:**
```html
<!-- Click event (default for buttons) -->
<button data-action="click->counter#increment">+</button>
<button data-action="counter#increment">+</button>

<!-- Input event (default for inputs) -->
<input data-action="input->search#query" type="text">
<input data-action="search#query" type="text">

<!-- Multiple actions -->
<input data-action="input->search#query focus->search#highlight">

<!-- Custom events -->
<div data-action="turbo:submit-end->form#handleResponse"></div>
```

**Default events by element:**
- `<button>`, `<a>`: `click`
- `<form>`: `submit`
- `<input>`, `<textarea>`, `<select>`: `input` or `change`

### Values

Values are typed data attributes that controllers can read and observe for changes.

**Defining values:**
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = {
    url: String,
    interval: Number,
    enabled: Boolean,
    items: Array,
    config: Object
  }
  
  connect() {
    console.log(this.urlValue)      // "https://example.com"
    console.log(this.intervalValue)  // 5000
    console.log(this.enabledValue)   // true
  }
}
```

**Using values in HTML:**
```html
<div data-controller="refresh"
     data-refresh-url-value="https://example.com"
     data-refresh-interval-value="5000"
     data-refresh-enabled-value="true">
</div>
```

**Value change callbacks:**
```javascript
export default class extends Controller {
  static values = { query: String }
  
  queryValueChanged(value, previousValue) {
    console.log(`Query changed from "${previousValue}" to "${value}"`)
    this.search(value)
  }
}
```

**Value accessors:**
- `this.queryValue` - Get the value
- `this.hasQueryValue` - Check if value exists
- Default values provided if attribute is missing

### Classes

Classes allow you to reference CSS class names in your controller without hardcoding them.

**Defining classes:**
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static classes = ["active", "hidden"]
  
  toggle() {
    this.element.classList.toggle(this.activeClass)
  }
  
  hide() {
    this.element.classList.add(this.hiddenClass)
  }
}
```

**Using classes in HTML:**
```html
<div data-controller="toggle"
     data-toggle-active-class="bg-blue-500"
     data-toggle-hidden-class="hidden">
</div>
```

**Multiple CSS classes:**

You can specify multiple CSS classes for a single logical name by separating them with spaces:
```html
<form data-controller="search"
      data-search-loading-class="bg-gray-500 animate-spinner cursor-busy">
  <input data-action="search#loadResults">
</form>
```

**Class accessors:**
- `this.activeClass` - Returns the first class name (if multiple, returns first)
- `this.activeClasses` - Returns array of all class names (split by spaces)
- `this.hasActiveClass` - Returns boolean if class attribute is defined

**Applying multiple classes:**
```javascript
export default class extends Controller {
  static classes = ["loading"]
  
  loadResults() {
    // Apply all classes at once using spread operator
    this.element.classList.add(...this.loadingClasses)
    
    fetch(/* ... */)
  }
  
  finishLoading() {
    // Remove all classes at once
    this.element.classList.remove(...this.loadingClasses)
  }
}
```

**Important:** CSS class attributes must be specified on the same element as the `data-controller` attribute.

### Params

Params allow you to pass additional data to action methods via `data-{controller}-{param}-param` attributes.

**Accessing params in actions:**
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  greet(event) {
    const name = event.params.name
    const greeting = event.params.greeting || "Hello"
    console.log(`${greeting}, ${name}!`)
  }
}
```

**Using params in HTML:**
```html
<div data-controller="greeter">
  <button data-action="greeter#greet"
          data-greeter-name-param="Alice"
          data-greeter-greeting-param="Hi">
    Greet Alice
  </button>
  
  <button data-action="greeter#greet"
          data-greeter-name-param="Bob">
    Greet Bob
  </button>
</div>
```

**Use cases:**
- Passing IDs or identifiers to actions
- Configuring behavior per element
- Avoiding data-value attributes for action-specific data

### Outlets

Outlets allow one controller to reference and communicate with another controller.

**Defining outlets:**
```javascript
// search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static outlets = ["results"]
  
  search() {
    const query = this.element.value
    this.resultsOutlet.update(query)
  }
}
```
```javascript
// results_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  update(query) {
    console.log(`Searching for: ${query}`)
    // Update results...
  }
}
```

**Using outlets in HTML:**
```html
<div data-controller="search"
     data-search-results-outlet="#search-results">
  <input type="text" data-action="search#search">
</div>

<div id="search-results" data-controller="results">
  <!-- Results appear here -->
</div>
```

**Outlet accessors:**
- `this.resultsOutlet` - Returns first matching outlet
- `this.resultsOutlets` - Returns array of all matching outlets
- `this.hasResultsOutlet` - Returns boolean if outlet exists

## Data Attributes Reference

### data-controller

Connects an element to a Stimulus controller:
```html
<div data-controller="dropdown">...</div>
<div data-controller="search results">...</div> <!-- Multiple controllers -->
```

### data-{controller}-target

Marks an element as a target within a controller:
```html
<input data-search-target="query" type="text">
<div data-search-target="results">...</div>
```

### data-action

Connects events to controller actions:
```html
<!-- Basic -->
<button data-action="click->counter#increment">+</button>

<!-- Default event -->
<button data-action="counter#increment">+</button>

<!-- Multiple actions -->
<input data-action="input->search#query focus->search#highlight">

<!-- Action options -->
<button data-action="click->modal#close:once">Close</button>
<div data-action="scroll->infinite#loadMore:passive">...</div>
```

**Action options:**
- `:once` - Fire action only once
- `:passive` - Don't call preventDefault
- `:capture` - Use capture phase
- `:stop` - Stop event propagation
- `:prevent` - Call preventDefault

### data-{controller}-{value}-value

Defines typed values for controllers:
```html
<div data-controller="slideshow"
     data-slideshow-index-value="0"
     data-slideshow-autoplay-value="true"
     data-slideshow-delay-value="3000">
</div>
```

### data-{controller}-{class}-class

Defines CSS classes for controllers:
```html
<div data-controller="toggle"
     data-toggle-active-class="bg-blue-500 text-white"
     data-toggle-inactive-class="bg-gray-200">
</div>
```

### data-{controller}-{param}-param

Passes parameters to action methods:
```html
<button data-action="modal#open"
        data-modal-size-param="large"
        data-modal-title-param="Settings">
  Open Modal
</button>
```

### data-{controller}-{outlet}-outlet

Connects controllers via outlets:
```html
<div data-controller="parent"
     data-parent-child-outlet="#child-element">
</div>
```

## Making Requests from Stimulus

### Using @rails/request.js

When making HTTP requests from Stimulus controllers to your Rails app, use the `@rails/request.js` library instead of raw `fetch`. This library automatically:
- Includes CSRF tokens for security
- Parses Turbo Stream responses automatically
- Handles Rails conventions out of the box

**Installation:**
```javascript
// Already included if using importmap-rails or esbuild-rails
import { post, get, patch, put, destroy } from "@rails/request.js"
```

**Basic usage:**
```javascript
import { Controller } from "@hotwired/stimulus"
import { post } from "@rails/request.js"

export default class extends Controller {
  async submit(event) {
    event.preventDefault()
    
    const formData = new FormData(event.target)
    
    const response = await post("/posts", {
      body: formData
    })
    
    if (response.ok) {
      // Success - Turbo Streams are automatically processed
      console.log("Post created!")
    } else {
      console.error("Failed to create post")
    }
  }
}
```

**Why use @rails/request.js:**
- **CSRF Protection**: Automatically includes `X-CSRF-Token` header
- **Turbo Stream Support**: Parses and applies `text/vnd.turbo-stream.html` responses
- **Rails Conventions**: Follows Rails HTTP patterns
- **Error Handling**: Proper response status handling

**Available methods:**
```javascript
import { get, post, patch, put, destroy } from "@rails/request.js"

// GET request
await get("/posts/1")

// POST request
await post("/posts", { body: formData })

// PATCH request
await patch("/posts/1", { body: formData })

// PUT request
await put("/posts/1", { body: formData })

// DELETE request
await destroy("/posts/1")
```

**Handling Turbo Stream responses:**
```javascript
import { Controller } from "@hotwired/stimulus"
import { post } from "@rails/request.js"

export default class extends Controller {
  async create(event) {
    event.preventDefault()
    
    const response = await post("/comments", {
      body: new FormData(event.target)
    })
    
    if (response.ok) {
      // Turbo Streams automatically applied to the page
      event.target.reset() // Clear the form
    }
  }
}
```

**When NOT to use @rails/request.js:**
- Making requests to external APIs (use standard `fetch()`)
- When you need full control over headers/options
- Non-Rails backends

## Common Patterns

### Form Enhancement

Add client-side validation and feedback:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "error"]
  
  validate() {
    const value = this.inputTarget.value
    
    if (value.length < 3) {
      this.showError("Must be at least 3 characters")
    } else {
      this.hideError()
    }
  }
  
  showError(message) {
    this.errorTarget.textContent = message
    this.errorTarget.classList.remove("hidden")
  }
  
  hideError() {
    this.errorTarget.classList.add("hidden")
  }
}
```
```html
<div data-controller="validation">
  <input data-validation-target="input"
         data-action="input->validation#validate"
         type="text">
  <p data-validation-target="error" class="hidden text-red-500"></p>
</div>
```

### Dynamic UI Updates

Toggle visibility, update content:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["content"]
  static classes = ["hidden"]
  
  toggle() {
    this.contentTarget.classList.toggle(this.hiddenClass)
  }
  
  show() {
    this.contentTarget.classList.remove(this.hiddenClass)
  }
  
  hide() {
    this.contentTarget.classList.add(this.hiddenClass)
  }
}
```
```html
<div data-controller="toggle"
     data-toggle-hidden-class="hidden">
  <button data-action="toggle#toggle">Toggle</button>
  <div data-toggle-target="content">
    Content to toggle
  </div>
</div>
```

### Debouncing Inputs

Prevent excessive API calls during typing:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = { delay: { type: Number, default: 300 } }
  
  search(event) {
    clearTimeout(this.timeout)
    
    this.timeout = setTimeout(() => {
      this.performSearch(event.target.value)
    }, this.delayValue)
  }
  
  performSearch(query) {
    console.log(`Searching for: ${query}`)
    // Make API call...
  }
  
  disconnect() {
    clearTimeout(this.timeout)
  }
}
```
```html
<div data-controller="search" data-search-delay-value="500">
  <input data-action="input->search#search" type="text" placeholder="Search...">
</div>
```

### Real-time Validation

Validate as user types:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["email", "feedback"]
  
  validate() {
    const email = this.emailTarget.value
    const isValid = this.isValidEmail(email)
    
    if (isValid) {
      this.showSuccess("Valid email!")
    } else if (email.length > 0) {
      this.showError("Invalid email format")
    } else {
      this.clearFeedback()
    }
  }
  
  isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  }
  
  showSuccess(message) {
    this.feedbackTarget.textContent = message
    this.feedbackTarget.className = "text-green-500"
  }
  
  showError(message) {
    this.feedbackTarget.textContent = message
    this.feedbackTarget.className = "text-red-500"
  }
  
  clearFeedback() {
    this.feedbackTarget.textContent = ""
  }
}
```
```html
<div data-controller="email-validation">
  <input data-email-validation-target="email"
         data-action="input->email-validation#validate"
         type="email">
  <p data-email-validation-target="feedback"></p>
</div>
```

### Toggling Visibility

Show/hide elements based on conditions:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["details"]
  static classes = ["hidden"]
  
  toggle() {
    this.detailsTarget.classList.toggle(this.hiddenClass)
  }
}
```
```html
<div data-controller="disclosure" data-disclosure-hidden-class="hidden">
  <button data-action="disclosure#toggle">
    Show Details
  </button>
  
  <div data-disclosure-target="details" class="hidden">
    <p>Additional details here...</p>
  </div>
</div>
```

## Lifecycle Callbacks

### Controller Callbacks

Controllers have lifecycle methods that fire at specific times:

**connect()**

Called when the controller is connected to the DOM:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    console.log("Controller connected!")
    // Initialize state, setup event listeners, etc.
  }
}
```

**Use cases:**
- Initialize controller state
- Set up third-party libraries
- Start intervals or timers
- Add global event listeners

**disconnect()**

Called when the controller is disconnected from the DOM:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    this.interval = setInterval(() => this.refresh(), 5000)
  }
  
  disconnect() {
    clearInterval(this.interval)
    console.log("Controller disconnected!")
  }
}
```

**Use cases:**
- Clean up intervals and timers
- Remove global event listeners
- Tear down third-party libraries
- Prevent memory leaks

### Target Callbacks

Controllers can respond when targets are added or removed:

**{target}TargetConnected()**

Called when a target is connected to the controller:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["item"]
  
  itemTargetConnected(element) {
    console.log("New item added:", element)
    element.classList.add("fade-in")
  }
}
```

**Use cases:**
- Initialize newly added targets
- Apply animations to new elements
- Set up target-specific event listeners

**{target}TargetDisconnected()**

Called when a target is disconnected from the controller:
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["item"]
  
  itemTargetDisconnected(element) {
    console.log("Item removed:", element)
    // Clean up resources for this target
  }
}
```

**Use cases:**
- Clean up target-specific resources
- Remove target-specific event listeners
- Track removed elements

**Example with both:**
```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["notification"]
  
  notificationTargetConnected(element) {
    // Auto-dismiss after 5 seconds
    element.dataset.timeout = setTimeout(() => {
      element.remove()
    }, 5000)
  }
  
  notificationTargetDisconnected(element) {
    // Clear timeout when manually dismissed
    clearTimeout(element.dataset.timeout)
  }
}
```

## Best Practices

### Keep Controllers Small and Focused

Each controller should do one thing well:
```javascript
// ✅ Good - focused controller
export default class extends Controller {
  toggle() {
    this.element.classList.toggle("hidden")
  }
}

// ❌ Bad - doing too much
export default class extends Controller {
  toggle() { /* ... */ }
  validate() { /* ... */ }
  submit() { /* ... */ }
  animate() { /* ... */ }
  // Split into separate controllers!
}
```

### Use Semantic Naming

Controller and action names should be clear and descriptive:
```javascript
// ✅ Good naming
data-controller="dropdown"
data-action="dropdown#toggle"

data-controller="search"
data-action="search#query"

// ❌ Bad naming
data-controller="thing"
data-action="thing#doIt"
```

### Progressive Enhancement

Ensure functionality works without JavaScript when possible:
```html
<!-- Form works without JavaScript -->
<form action="/search" method="get" data-controller="search">
  <input name="q" 
         type="text"
         data-action="input->search#liveSearch">
  <button type="submit">Search</button>
</form>
```

The form submits normally, but Stimulus adds live search as an enhancement.

### Use Values for Configuration

Don't hardcode values in controllers:
```javascript
// ✅ Good - configurable
export default class extends Controller {
  static values = { interval: { type: Number, default: 5000 } }
  
  connect() {
    setInterval(() => this.refresh(), this.intervalValue)
  }
}

// ❌ Bad - hardcoded
export default class extends Controller {
  connect() {
    setInterval(() => this.refresh(), 5000)
  }
}
```

### Clean Up in disconnect()

Always clean up resources to prevent memory leaks:
```javascript
export default class extends Controller {
  connect() {
    this.interval = setInterval(() => this.tick(), 1000)
    this.handleResize = () => this.resize()
    window.addEventListener("resize", this.handleResize)
  }
  
  disconnect() {
    clearInterval(this.interval)
    window.removeEventListener("resize", this.handleResize)
  }
}
```

### Leverage Existing HTML Structure

Work with server-rendered HTML instead of building DOM from JavaScript:
```javascript
// ✅ Good - enhance existing HTML
export default class extends Controller {
  connect() {
    this.element.querySelectorAll(".item").forEach(item => {
      item.classList.add("enhanced")
    })
  }
}

// ❌ Bad - building HTML in JavaScript
export default class extends Controller {
  connect() {
    this.element.innerHTML = `
      <div class="item enhanced">...</div>
      <div class="item enhanced">...</div>
    `
  }
}
```

## Troubleshooting

### Controller Not Connecting

**Problem:** Controller's `connect()` method never fires.

**Common Causes:**
1. Controller file not imported/registered
2. Typo in controller name
3. JavaScript errors preventing execution

**Solution:**
```javascript
// 1. Verify controller is imported
// app/javascript/controllers/index.js
import HelloController from "./hello_controller"
application.register("hello", HelloController)

// 2. Check for typos
// ❌ Wrong
<div data-controller="helo">

// ✅ Correct
<div data-controller="hello">

// 3. Check browser console for errors
```

### Actions Not Firing

**Problem:** Clicking/interacting with element doesn't trigger action.

**Common Causes:**
1. Incorrect action syntax
2. Method name doesn't match
3. Event not matching element type

**Solution:**
```html
<!-- ❌ Wrong syntax -->
<button data-action="hello.greet">Click</button>

<!-- ✅ Correct syntax -->
<button data-action="hello#greet">Click</button>

<!-- ❌ Wrong event for element -->
<button data-action="submit->hello#greet">Click</button>

<!-- ✅ Correct event -->
<button data-action="click->hello#greet">Click</button>
<!-- OR use default -->
<button data-action="hello#greet">Click</button>
```

### Target Not Found

**Problem:** Error: "Missing target element" or `this.nameTarget` is undefined.

**Cause:** Target element doesn't exist or has wrong attribute.

**Solution:**
```html
<!-- ❌ Wrong - typo in target name -->
<div data-hello-target="nam">Name</div>

<!-- ✅ Correct -->
<div data-hello-target="name">Name</div>
```

**Use optional target checking:**
```javascript
// Instead of this (throws error if missing):
this.nameTarget.textContent = "Hello"

// Use this (safe):
if (this.hasNameTarget) {
  this.nameTarget.textContent = "Hello"
}
```

### Controller State Lost After Turbo Navigation

**Problem:** Controller state resets after Turbo Drive navigation.

**Cause:** Turbo Drive replaces the body, disconnecting and reconnecting controllers.

**Solution:** Store state in data attributes or use Turbo's cache events:
```javascript
export default class extends Controller {
  static values = { count: Number }
  
  increment() {
    this.countValue++  // Automatically persists to data attribute
  }
  
  // State is preserved in HTML and restored on reconnect
}
```

### Memory Leaks

**Problem:** Application slows down over time, especially with Turbo navigation.

**Cause:** Not cleaning up event listeners, intervals, or timers.

**Solution:**
```javascript
export default class extends Controller {
  connect() {
    this.interval = setInterval(() => this.refresh(), 1000)
    this.handleClick = (e) => this.onClick(e)
    document.addEventListener("click", this.handleClick)
  }
  
  disconnect() {
    // ✅ Always clean up!
    clearInterval(this.interval)
    document.removeEventListener("click", this.handleClick)
  }
}
```

### Values Not Updating

**Problem:** Value changes don't trigger `{value}ValueChanged` callback.

**Common Causes:**
1. Value not properly defined in `static values`
2. Callback method name doesn't match convention
3. Manually setting data attribute instead of using value property

**Solution:**
```javascript
// ✅ Correct - this DOES trigger callback and update attribute
this.countValue = 10

// ❌ Wrong - manually setting attribute bypasses Stimulus
this.element.dataset.controllerCountValue = 10  // Don't do this
this.element.setAttribute("data-controller-count-value", 10)  // Don't do this
```

**Verify your setup:**
```javascript
export default class extends Controller {
  // 1. Ensure value is defined
  static values = { count: Number }
  
  // 2. Ensure callback name matches exactly
  countValueChanged(value, previousValue) {
    console.log(`Count changed from ${previousValue} to ${value}`)
  }
  
  increment() {
    // 3. Always use the value property, not the dataset
    this.countValue++  // ✅ Correct
  }
}
```

**If callback still isn't firing:**
- Check browser console for JavaScript errors
- Verify the value type matches (Number, String, Boolean, Array, Object)
- Ensure you're not preventing the update elsewhere in your code

## Additional Resources

- [Stimulus Handbook](https://stimulus.hotwired.dev/handbook/introduction)
- [Stimulus Reference](https://stimulus.hotwired.dev/reference/controllers)
- [Stimulus Best Practices](https://stimulus.hotwired.dev/handbook/managing-state)