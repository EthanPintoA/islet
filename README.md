# Islet

_A front-end methodology for building modular, native-first web interfaces_

> **🚧 Work in Progress**
>
> This methodology is still being developed. The documentation is incomplete, and the methods are subject to change.

```javascript
/** @readonly */
/*Is*/let ethos = "minimal & modular";
```

Islet is a front-end methodology that treats the web platform itself as the primary framework. It aims to minimize abstractions by relying on native web technologies (HTML, CSS, JS) and Web APIs instead of heavy abstractions like large frameworks or libraries.

The methodology focuses on reducing unnecessary tooling while preserving a strong developer experience. It introduces conventions that make codebases scalable without framework lock-in, helping applications stay lightweight, performant, and backward-compatible as standards evolve.

To support this, the approach incorporates a set of methods, which are the mechanisms that enforce modularity by convention while minimizing complexity and dependencies.

## Methods

### Islands

Islet promotes the use of **Islands** which are inspired by the Islands Architecture pattern. Islands are dynamic components that encapsulate behavior and state for a specific part of the UI. Unlike traditional components, islands defined in JavaScript are not self-contained units; instead, they enhance static HTML structures with interactivity.

#### Defining Islands

Islands utilize [Custom Elements](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements) to define their behavior, allowing them to easily integrate with server-rendered HTML.

```html
<script type="module">
  class Alerter extends HTMLElement {
    connectedCallback() {
      this.querySelector("button").addEventListener("click", () => {
        alert(this.dataset.alertMessage);
      });
    }
  }
  customElements.define("alerter-component", Alerter);
</script>

<alerter-component data-alert-message="Hello World!">
  <button>Hello!</button>
</alerter-component>
```

### State Management

> [!CAUTION]
>
> It's not recommended to use prop passing or dispatching events directly between islands since custom elements are not guaranteed to be "upgraded" (have their constructor called) from top to bottom. Using a shared event bus or state management solution ensures reliable communication between islands regardless of their upgrade order.
>
> Since islands are not tightly coupled to HTML, you should be able to have custom elements wrap around larger sections of the DOM that share state without worrying about prop drilling or event propagation.

Islands are independent, but they often need to talk. Islet supports two primary patterns:

#### 1. Native Event Bus or Data Store (Recommended)

Use `EventTarget` with `Event` and `CustomEvent` to create lightweight stores without external dependencies.

<details>

<summary>View Native Example</summary>

```html
<script type="module">
  class MessageStore extends EventTarget {
    message = "";

    setMessage(message) {
      this.message = message;
      this.dispatchEvent(new CustomEvent("change", { detail: { message } }));
    }
  }
  const messageStore = new MessageStore();

  class ProducerIsland extends HTMLElement {
    connectedCallback() {
      this.querySelector("input").addEventListener("input", () => {
        messageStore.setMessage(event.target.value);
      });
    }
  }
  customElements.define("producer-island", ProducerIsland);

  class ConsumerIsland extends HTMLElement {
    connectedCallback() {
      messageStore.addEventListener("change", (event) => {
        this.querySelector("p").textContent = event.detail.message;
      });
    }
  }
  customElements.define("consumer-island", ConsumerIsland);
</script>

<producer-island>
  <input type="text" placeholder="Type a message" />
</producer-island>
<consumer-island>
  <p>No message yet.</p>
</consumer-island>
```

</details>

#### 2. External State Management Libraries (Optional)

Developers are free to plug in libraries like [Preact Signals Core](https://github.com/preactjs/signals) or [Nano Stores](https://github.com/nanostores/nanostores).

<details>

<summary>View Preact Signals Core Example</summary>

```html
<script type="module">
  import { signal, effect } from "@preact/signals-core";

  const messageSignal = signal("No message yet.");

  class ProducerIsland extends HTMLElement {
    connectedCallback() {
      this.querySelector("input").addEventListener("input", (event) => {
        messageSignal.value = event.target.value;
      });
    }
  }
  customElements.define("producer-island", ProducerIsland);

  class ConsumerIsland extends HTMLElement {
    connectedCallback() {
      effect(() => (this.querySelector("p").textContent = messageSignal.value));
    }
  }
  customElements.define("consumer-island", ConsumerIsland);
</script>

<producer-island>
  <input type="text" placeholder="Type a message" />
</producer-island>
<consumer-island>
  <p>No message yet.</p>
</consumer-island>
```

</details>

### CSS Styling

Islet emphasizes native CSS for styling, avoiding CSS-in-JS or other abstractions.

[RSCSS (Reasonable System for CSS Stylesheet Structure)](https://rstacruz.github.io/rscss/) is used as a guideline for writing maintainable and scalable CSS. RSCSS is similar to BEM but leverages cascading and specificity rather than avoiding them.

A few adjustments are made to RSCSS for Islet, including:

- Islands are treated as RSCSS "components."
- Styling islands should be done with element selectors instead of class names since custom elements provide unique selectors.

### Style Organization

Islet encourages using [ITCSS (Inverted Triangle CSS)](https://www.creativebloq.com/web-design/manage-large-css-projects-itcss-101517528) to structure stylesheets. This allows developers to keep styles organized and leverage the benefits of cascading.

The ITCSS approach organizes styles into layers, starting from the most generic styles at the base and moving towards more specific styles at the tip. This structure helps manage specificity and maintainability as applications grow.

### RESTful HTML APIs

Inspired by [htmx](https://htmx.org/), Islet prefers using hypermedia-rich HTML over JSON for RESTful server-client communication.

**Why this approach?**

- Leverages Server-Side Templating: Handles dynamic content generation, allowing the server to send HTML fragments that can be directly injected into the DOM.
- Progressive Enhancement: Ensures functionality even without JavaScript.
- Seamless Integration: Since our [islands](#islands) utilize custom elements, any server-sent HTML that contains these elements will automatically gain their behavior upon insertion into the DOM.

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

**No License / All Rights Reserved**

You may not reuse, copy, modify, or redistribute the documentation or code at this time.

Open Source licenses for the website, documentation, and code examples/samples will be will be applied upon the release of the first stable draft.
