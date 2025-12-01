# Islet

_A front-end methodology for building modular, native-first web interfaces_

> **🚧 Work in Progress**
>
> This methodology is still being developed. The documentation is incomplete, and the methods are subject to change.

## Overview

```javascript
/** @readonly */
/*Is*/let ethos = "minimal & modular";
```

Islet is a front-end methodology that treats the web platform itself as the primary framework and aims to minimize abstractions by relying on native web technologies. It encourages developers to use HTML, CSS, JavaScript, and Web APIs directly to build modular, maintainable applications instead of relying on heavy abstractions such as large frameworks or libraries.

The methodology focuses on reducing unnecessary tooling while preserving a strong developer experience. It introduces conventions that make codebases scalable without framework lock-in, helping applications stay lightweight, performant, and backward-compatible as standards evolve.

To support this, the approach incorporates a set of methods which are all chosen to enforce modularity by convention. They are the mechanisms that allow applications to take advantage of browsers' strengths while minimizing complexity and be more lightweight.

## Methods

### Islands

Islet promotes the use of "islands" which are inspired by the islands architecture pattern. Islands are dynamic components that encapsulate the functionality and state of a specific part of the user interface. Each island is **not** self-contained like web components or frameworks' components; instead, islands solely encapsulate behavior and internal state in the scope of its element and static children.

Islands are created using custom elements from the Web Components Suite, but they do not rely on the shadow DOM. This allows islands to be more lightweight and easier to style using global CSS while still benefiting from the reusability provided by custom elements.

```html
<script type="module">
  class MyIsland extends HTMLElement {
    connectedCallback() {
      this.querySelector("button").addEventListener("click", () => {
        alert(this.dataset.alertMsg);
      });
    }
  }
  customElements.define("my-island", MyIsland);
</script>
<my-island data-alert-msg="Hello World!">
  <button>Hello!</button>
</my-island>
<my-island data-alert-msg="Hello Universe!">
  <button>Hello!!!</button>
</my-island>
```

#### Data Sharing Between Islands

Islet encourages the use of native browser features for data sharing between islands. This can be achieved by using `EventTarget` instances and `Event` or `CustomEvent` objects to act as event buses and data stores.

```html
<script type="module">
  class MessageStore extends EventTarget {
    message = "";

    setMessage(message) {
      this.message = message;
      const event = new CustomEvent("change", { detail: { message } });
      this.dispatchEvent(event);
    }
  }
  const messageStore = new MessageStore();

  class ProducerIsland extends HTMLElement {
    connectedCallback() {
      this.querySelector("button").addEventListener("click", () => {
        messageStore.setMessage(this.querySelector("input").value);
      });
    }
  }
  customElements.define("producer-island", ProducerIsland);

  class ConsumerIsland extends HTMLElement {
    connectedCallback() {
      const msgHolder = this.querySelector("[data-js-message-holder]");
      messageStore.addEventListener("change", (event) => {
        msgHolder.textContent = event.detail.message;
      });
    }
  }
  customElements.define("consumer-island", ConsumerIsland);
</script>
<producer-island>
  <input type="text" placeholder="Type a message" />
  <button>Send Message</button>
</producer-island>
<consumer-island>
  <p data-js-message-holder>No message yet.</p>
</consumer-island>
```

However, developers are free to use or implement other data-sharing mechanisms as needed. For example, they can use libraries like [Preact Signals Core](https://github.com/preactjs/signals/tree/main/packages/core) or [Nano Stores](https://github.com/nanostores/nanostores) to manage state and share data between islands.

```html
<script type="module">
  import { signal } from "@preact/signals-core";

  const messageSignal = signal("No message yet.");

  class ProducerIsland extends HTMLElement {
    connectedCallback() {
      this.querySelector("button").addEventListener("click", () => {
        messageSignal.value = this.querySelector("input").value;
      });
    }
  }
  customElements.define("producer-island", ProducerIsland);

  class ConsumerIsland extends HTMLElement {
    connectedCallback() {
      messageSignal.subscribe((newValue) => {
        this.querySelector("p").textContent = newValue;
      });
    }
  }
  customElements.define("consumer-island", ConsumerIsland);
</script>
<producer-island>
  <input type="text" placeholder="Type a message" />
  <button>Send Message</button>
</producer-island>
<consumer-island>
  <p>No message yet.</p>
</consumer-island>
```

> [!CAUTION]
>
> It's not recommended to use prop passing since custom elements are not guaranteed to be "upgraded" (have their constructor called) from top to bottom. Islands should be treated as independent units that communicate through shared state or events, even if they are nested within each other.

### Styling

Islet encourages the use of native CSS for styling islands and the overall application. To promote modularity, [RSCSS (Reasonable System for CSS Stylesheet Structure)](https://rstacruz.github.io/rscss/) is used as a guideline for writing maintainable and scalable CSS. RSCSS is very similar to BEM but takes advantage of cascading and specificity rather than treating them as problems to be avoided.

This methodology's approach to styling slightly differs from RSCSS, such as encouraging the use of element selectors for [islands](#islands) instead of class names given that custom elements already provide unique selectors. Islands should be considered RSCSS "components," and their styles should be scoped accordingly.

### Style Organization

In order to keep styles organized and modular, Islet suggests the usage of [ITCSS (Inverted Triangle CSS)](https://www.creativebloq.com/web-design/manage-large-css-projects-itcss-101517528) as a methodology for structuring stylesheets.

The ITCSS approach organizes styles into layers, starting from the most generic styles at the base and moving towards more specific styles at the top. This helps to manage specificity and maintainability as the application grows.

### RESTful HTML APIs

We follow RESTful principles using HTML as the primary media type, an approach inspired by [htmx](https://htmx.org/). Instead of JSON, the server returns hypermedia-rich HTML fragments that are swapped directly into the DOM.

**Why this approach?**

- Simplicity: Reduces client-side rendering logic and state management.
- Performance: Minimizes JavaScript execution and bundle size by avoiding rendering on the client.
- Progressive Enhancement: Ensures functionality even without JavaScript.
- Seamless Integration: Since our [islands](#islands) utilize custom elements, any server-sent HTML that contains these elements will automatically gain their behavior upon insertion into the DOM.
- Leverages Server-Side Templating: Allows easy generation of dynamic content using server-side templating languages. Also, full-pages and fragments can share templating logic, reducing duplication.

> [!NOTE]
>
> HTML is the default, but JSON is still acceptable for specific use cases. If you need to transfer large amounts of structured data, use a JSON API. For minor data needs within HTML, utilize `data-*` attributes or embedded script tags (`<script type="application/json">`).

## Issues, Questions, and Contributions

At this time:

🚫 Issues are not being accepted\
🚫 Questions will not be addressed\
🚫 Contributions are not open

Once licensing and documentation stabilize, contribution and issues will be welcomed.

## License

No license has been applied yet. You may not reuse, copy, modify, or redistribute the documentation or code examples at this time.

Licenses for the website, documentation, and example code will be added once the project reaches a stable draft.
