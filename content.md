# Managing JavaScript Page Lifecycle Events 🔂
In this lesson, we'll explore how to manage JavaScript page lifecycle events within a Ruby on Rails application. These lifecycle events are crucial for controlling when and how your JavaScript interacts with your web page, especially in modern Rails apps that often use tools like [Turbo Drive](https://turbo.hotwired.dev/) and [Stimulus](https://stimulus.hotwired.dev/) to enhance performance and user experience.

## Introduction to JavaScript Page Lifecycle Events
JavaScript page lifecycle events help you manage your web page's behavior during different stages of loading and user interaction. Understanding these events allows you to execute JavaScript code at the right moments, making your app more dynamic and responsive. If you're writing javascript (or using a library) that acts upon one of your HTML elements: you need to wait until the element that you want to call the method on has been loaded before it will work.

<!-- TODO: provide practical use cases for using these event listeners -->

### Full Page Loads
When a web page fully loads in a browser, several key events occur. Here are the most important ones:

#### DOMContentLoaded
This event fires when the initial HTML document is fully loaded and parsed, without waiting for stylesheets, images, or other external resources. It’s the perfect time to initialize your application logic.

```html
<script>
  document.addEventListener('DOMContentLoaded', function() {
    console.log('DOM fully loaded and parsed');
  });
</script>
```

#### load
This event fires when the entire page, including all dependent resources like images and stylesheets, has finished loading.

```html
<script>
  window.addEventListener('load', function() {
    console.log('Page fully loaded');
  });
</script>
```

## Using Turbo Drive in Rails
In modern Rails applications, Turbo Drive (part of [Hotwire](https://hotwired.dev/)) replaces full page reloads with partial updates. It enhances user experience by making navigation faster and smoother.

### Key Turbo Drive Events

#### turbo:load
Similar to `DOMContentLoaded`, but it triggers every time a new page is loaded by Turbo Drive. It’s the best place to run JavaScript that should execute every time a new page is loaded.

```html
<script>
  document.addEventListener('turbo:load', function() {
    console.log('Turbo Drive page load');
  });
</script>
```

#### turbo:before-cache
This event fires before Turbo Drive saves the current page to its cache. You can use it to clean up any state that should not be preserved between pages.

```html
<script>
  document.addEventListener('turbo:before-cache', function() {
    console.log('Preparing for Turbo cache');
  });
</script>
```

## Using Stimulus for Page Lifecycle Management
[Stimulus](https://stimulus.hotwired.dev/) is a JavaScript framework that works well with [Turbo Drive](https://turbo.hotwired.dev/), providing a structured way to manage JavaScript behavior tied to specific HTML elements. It makes it easy to handle dynamic interactions within your Rails app.

### Example Stimulus Controller

#### HTML
Attach a Stimulus controller to an element using the data-controller attribute.

```html
<div data-controller="example">
  <p data-example-target="text">Hello, Stimulus!</p>
</div>
```

#### JavaScript
Define the behavior in a Stimulus controller file.

```bash
rails generate stimulus example
```

This command generates a new JavaScript file `example_controller.js` in the `app/javascript/controllers` directory and automatically registers it in `index.js`.

```markdown
app/
└── javascript/
    └── controllers/
        ├── example_controller.js
        ├── index.js
```

```javascript
import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="example"
export default class extends Controller {
  static targets = [ "text" ];

  connect() {
    console.log('Stimulus controller connected');
    console.log('Waiting 2 seconds')
    setTimeout(() => this.setText(), 2000);
  }

  setText() {
    console.log('Setting text');
    this.textTarget.textContent = 'Stimulus is connected!';
  }

  disconnect() {
    console.log('Stimulus controller disconnected');
  }
}

```

![](assets/stimulus-example-1.gif)

In this example, the connect method runs when the element is added to the DOM, and the disconnect method runs when it’s removed.

## Using Stimulus to Filter a List
To illustrate how Stimulus can be used, let's walk through an example where we filter a list of items based on user input.

![](assets/stimulus-example-2.gif)

### Step 1: Create the Stimulus Controller
Rails provides a convenient generator for creating Stimulus controllers. To generate a new filter controller, run the following command in your terminal:

```bash
rails generate stimulus filter
```

This command generates a new JavaScript file `filter_controller.js` in the `app/javascript/controllers` directory and automatically registers it in `index.js`.

### Step 2: Set Up HTML with Stimulus Attributes
In your view file (e.g., `app/views/misc/home.html.erb`), add the following HTML. This HTML uses Stimulus attributes to bind elements to the controller:

```html
<div data-controller="filter">
  <input type="text" data-action="input->filter#applyFilters" placeholder="Search items">
  <ul data-filter-target="list">
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
  </ul>
</div>
```

#### Explanation:

- `data-controller="filter"`: Attaches the filter Stimulus controller to the `div`.
- `data-action="input->filter#applyFilters"`: Binds the input event on the text field to the applyFilters method in the filter controller. `input` specifies the event type to listen for (in this case, the input event which occurs when the user types in the field). `filter#applyFilters` specifies the method (`applyFilters`) in the filter controller to call when the event occurs.
- `data-filter-target="list"`: Marks the `ul` element as a target that the filter controller can reference directly.

### Step 3: Implement the Controller Logic
Edit the generated `filter_controller.js` file to add the behavior for filtering the list:

```javascript
// app/javascript/controllers/filter_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "list" ];

  applyFilters(event) {
    console.log("applying filter");
    console.log(event);
    const query = event.target.value.toLowerCase();
    this.listTarget.querySelectorAll('li').forEach(item => {
      item.style.display = item.textContent.toLowerCase().includes(query) ? '' : 'none';
    });
    console.log("done");
  }
}
```

#### Explanation:

- `static targets = [ "list" ];`: Defines list as a target, making it easily accessible in the controller.
- `applyFilters(event)`: This method is triggered by the input event on the text field. It filters the list items based on the input value by showing or hiding items.

Stimulus provides a structured way to add JavaScript behavior to your Rails application. Using Stimulus generators, you can quickly set up controllers, and with attributes like `data-controller` and `data-action`, you can easily bind JavaScript functionality to your HTML elements. This approach helps you keep your JavaScript organized and closely tied to the elements it interacts with, leading to cleaner and more maintainable code.

## Quiz

- What event is triggered when the initial HTML document has been completely loaded and parsed?
- `load`
  - Not quite. This event fires after the entire page, including images and stylesheets, is fully loaded.
- `DOMContentLoaded`
  - Correct! `DOMContentLoaded` fires when the initial HTML document is fully loaded and parsed.
- `turbo:load`
  - Not quite. turbo:load is specific to Turbo Drive and fires after a page load or change.
{: .choose_best #question1 title="DOMContentLoaded Event" points="1" answer="2" }

- What does `data-action="input->filter#applyFilters"` specify in a Stimulus controller?
- It calls the `applyFilters` method whenever the input event occurs.
  - Correct! This attribute connects the input event to the `applyFilters` method in the filter controller.
- It binds the `applyFilters` method to the input tag.
  - Not quite. While it involves the input event, it doesn't directly bind to the input tag.
- It sets a default action for the filter controller.
  - Not quite. It specifies an event-action mapping, not a default action.
{: .choose_best #question2 title="Stimulus data-action" points="1" answer="1" }

- How does the connect method in a Stimulus controller differ from `DOMContentLoaded`?
- `connect` is triggered once when the initial page loads.
  - Not exactly. connect can be triggered multiple times as elements are added or removed from the DOM.
- `connect` runs every time the controller’s element is connected to the DOM.
  - Correct! connect is called each time the element is added to the DOM, making it useful for dynamic content.
- `connect` is called after all resources on the page have loaded.
  - Not quite. This is what the load event does.
{: .choose_best #question3 title="Stimulus connect method" points="1" answer="2" }

- Why would you want to use Stimulus in a Rails application?
- To organize and manage JavaScript behavior in a structured way.
  - Correct! Stimulus helps keep JavaScript behavior organized and tied to specific elements, making the code easier to maintain.
- To replace Turbo Drive for handling AJAX requests.
  - Not quite. Turbo Drive and Stimulus serve different purposes. Turbo Drive handles partial page updates, while Stimulus organizes JavaScript behavior.
- To avoid using any JavaScript in the application.
  - Not exactly. Stimulus is used to enhance JavaScript management, not to avoid it.
{: .choose_best #question4 title="Benefits of Stimulus" points="1" answer="1" }

- What does the `data-controller` attribute do in a Stimulus-powered Rails application?
- It attaches a Stimulus controller to the element.
  - Correct! The `data-controller` attribute connects the specified controller to the element and its children.
- It specifies an action for the element.
  - Not quite. This is handled by the `data-action` attribute.
- It marks the element as a target in the controller.
  - Not exactly. This is done with the `data-target` attribute.
{: .choose_best #question5 title="Stimulus data-controller" points="1" answer="1" }

- Where are Stimulus controller files typically stored in a Rails application?
- `app/controllers`
  - Not quite. This directory is for Rails server-side controllers.
- `app/assets/javascripts`
  - Not exactly. This directory is for older Rails versions' JavaScript assets.
- `app/javascript/controllers`
  - Correct! This directory holds all the Stimulus controller files.
{: .choose_best #question6 title="Stimulus Controller File Location" points="1" answer="3" }

## Conclusion
Managing JavaScript page lifecycle events effectively can greatly enhance the performance and user experience of your applications. By mastering these concepts, you'll be well-equipped to manage the lifecycle of your JavaScript, making your applications more interactive and user-friendly.

## Resources
- [How do JavaScript Page Lifecycle Events work?](https://tilomitra.com/html-page-lifecycle-events/)
- [Stimulus](https://github.com/hotwired/stimulus)
- [Turbo](https://github.com/hotwired/turbo-rails)
