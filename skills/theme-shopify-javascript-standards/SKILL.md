---
name: theme-shopify-javascript-standards
description: JavaScript standards for Shopify themes - custom elements, file structure, and best practices. Use when writing JavaScript for Shopify theme sections.
---

# Shopify JavaScript Standards

JavaScript file structure, custom elements, and coding standards for Shopify theme development.

## When to Use

- Writing JavaScript for Shopify theme sections
- Creating interactive components
- Setting up JavaScript file structure
- Using custom HTML elements

## File Structure

### Separate JavaScript Files

- JavaScript must live in a **separate file** in the `assets/` directory
- Include it in the section using the `asset_url` filter

### Including JavaScript in Sections

```liquid
<script src="{{ 'section-logic.js' | asset_url }}" defer="defer"></script>
```

**Always use `defer`** to ensure scripts load after HTML parsing.

### File Naming

Match JavaScript file names to section names:

```
sections/
  └── product-quick-view.liquid

assets/
  └── product-quick-view.js
```

## JavaScript Rules

### Variable Declarations

- Use only `const` and `let`
- **Never use `var`**
- Avoid global scope pollution

### Example

```javascript
// Good
const productId = document.querySelector('[data-product-id]').dataset.productId;
let isOpen = false;

// Bad
var productId = ...; // Never use var
window.myVariable = ...; // Avoid global pollution
```

### Element Selection: Data Attributes Only

- **Never** select elements by class (e.g. `querySelector('.product-card')`)
- **Always** use data attributes: `querySelector('[data-product-card]')`, `querySelector('[data-trigger]')`
- Keeps JS independent from CSS class names and avoids breakage when styles change

### Scope Management

Keep variables scoped to their usage:

```javascript
// Good - scoped within function, selectors via data attributes
function initProductCard() {
  const card = document.querySelector('[data-product-card]');
  const button = card?.querySelector('[data-product-card-button]');
  // ...
}

// Bad - global variables, class selectors
const card = document.querySelector('.product-card'); // Never use class selectors
```

## Custom Elements

### Use Custom HTML Elements

Use **custom HTML elements** to encapsulate JavaScript logic and create reusable components.

### Custom Element Structure

```javascript
if (!customElements.get('product-quick-view')) {
  class ProductQuickView extends HTMLElement {
    constructor() {
      super();
    }

    connectedCallback() {
      this.setProperties();
      this.setEventListeners();
    }

    disconnectedCallback() {
      this.removeEventListeners();
    }

    setProperties() {
      this.button = this.querySelector('[data-trigger]');
      this.modal = this.querySelector('[data-modal]');
      this.closeButton = this.querySelector('[data-close]');
    }

    setEventListeners() {
      this.handleOpen = this.open.bind(this);
      this.handleClose = this.close.bind(this);
      this.button?.addEventListener('click', this.handleOpen);
      this.closeButton?.addEventListener('click', this.handleClose);
    }

    removeEventListeners() {
      this.button?.removeEventListener('click', this.handleOpen);
      this.closeButton?.removeEventListener('click', this.handleClose);
    }

    open() {
      this.modal?.classList.add('is-open');
    }

    close() {
      this.modal?.classList.remove('is-open');
    }
  }

  customElements.define('product-quick-view', ProductQuickView);
}
```

### Using Custom Elements in Liquid

```liquid
<product-quick-view data-product-id="{{ product.id }}">
  <button data-trigger>Quick View</button>
  <div data-modal class="modal">
    <button data-close>Close</button>
    <!-- Modal content -->
  </div>
</product-quick-view>
```

### Custom Element Benefits

- Encapsulation - logic is self-contained
- Reusability - use anywhere in the theme
- Lifecycle hooks - `connectedCallback`, `disconnectedCallback`
- Data attributes - pass data via `data-*` attributes

### Lifecycle Hooks

Use `connectedCallback` for setup and `disconnectedCallback` for cleanup. Split logic into `setProperties()`, `setEventListeners()`, and `removeEventListeners()`:

```javascript
if (!customElements.get('my-component')) {
  class MyComponent extends HTMLElement {
    constructor() {
      super();
    }

    connectedCallback() {
      this.setProperties();
      this.setEventListeners();
    }

    disconnectedCallback() {
      this.removeEventListeners();
    }

    setProperties() {
      // Cache element references
    }

    setEventListeners() {
      // Add listeners (e.g. this.handleOpen = this.open.bind(this); convention: handle + Event)
    }

    removeEventListeners() {
      // Remove all listeners
    }
  }

  customElements.define('my-component', MyComponent);
}
```

## Data Attributes

### Passing Data to JavaScript

- Use custom HTML tags when appropriate
- Pass dynamic data via `data-*` attributes
- Access data via `dataset` property

### Example

```liquid
<product-card 
  data-product-id="{{ product.id }}"
  data-product-handle="{{ product.handle }}"
  data-variant-id="{{ product.selected_or_first_available_variant.id }}">
  <!-- Content -->
</product-card>
```

```javascript
class ProductCard extends HTMLElement {
  constructor() {
    super();
    this.productId = this.dataset.productId;
    this.productHandle = this.dataset.productHandle;
    this.variantId = this.dataset.variantId;
  }
}
```

## Observers

### Use Observers Only When Explicitly Requested

Use observers (IntersectionObserver, MutationObserver, etc.) **only if explicitly requested** by the user.

### IntersectionObserver Example

```javascript
// Only use if explicitly needed
class LazyImage extends HTMLElement {
  constructor() {
    super();
    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.handleLoadImage();
          this.observer.unobserve(this);
        }
      });
    });
  }

  connectedCallback() {
    this.observer.observe(this);
  }

  handleLoadImage() {
    const img = this.querySelector('img');
    img.src = img.dataset.src;
  }
}
```

## Best Practices

### Event Handling

```javascript
class ProductForm extends HTMLElement {
  constructor() {
    super();
    this.form = this.querySelector('form');
    this.handleSubmit = this.submit.bind(this);
    this.form?.addEventListener('submit', this.handleSubmit);
  }

  submit(event) {
    event.preventDefault();
    // Handle form submission
  }

  disconnectedCallback() {
    // Clean up event listeners
    this.form?.removeEventListener('submit', this.handleSubmit);
  }
}
```

### Error Handling

```javascript
class ProductCard extends HTMLElement {
  init() {
    try {
      const button = this.querySelector('[data-add-to-cart]');
      if (!button) {
        console.warn('Add to cart button not found');
        return;
      }
      this.handleAddToCart = this.addToCart.bind(this);
      button.addEventListener('click', this.handleAddToCart);
    } catch (error) {
      console.error('Error initializing product card:', error);
    }
  }
}
```

### Styling

- **Never** set styles directly in JavaScript (`element.style.display = 'none'`)
- **Always** add/remove CSS classes (`element.classList.add('is-open')`, `classList.remove()`)
- Keep visual state in CSS; JS only toggles classes
### Inter-component Communication

- Use **CustomEvent** when components need to talk to each other or to parent/sections
- Dispatch from the component; listen on `document` or a common ancestor if needed

```javascript
this.dispatchEvent(new CustomEvent('cart:updated', { bubbles: true, detail: { count: 1 } }));
```
## Shopify Theme Documentation

Reference these official Shopify resources:

- [Shopify Theme JavaScript](https://shopify.dev/docs/themes/architecture/theme-assets)
- [Web Components](https://developer.mozilla.org/en-US/docs/Web/Web_Components)
- [Custom Elements](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)
- [Data Attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/data-*)

## Complete Example

### Section File

```liquid
{{ 'product-card.css' | asset_url | stylesheet_tag }}

<product-card 
  data-product-id="{{ product.id }}"
  data-product-handle="{{ product.handle }}">
  <div class="product-card">
    {{ image | image_tag: widths: '360, 720, 1080', loading: 'lazy' }}
    <h3>{{ product.title }}</h3>
    <button data-add-to-cart>Add to Cart</button>
  </div>
</product-card>

<script src="{{ 'product-card.js' | asset_url }}" defer="defer"></script>
```

### JavaScript File
All interactive components **must** use Web Components wrapped in IIFE:
```javascript
(()=>{
  if (!customElements.get('product-card')) {
    class ProductCard extends HTMLElement {
      constructor() {
        super();
      }

      connectedCallback() {
        this.setProperties();
        this.setEventListeners();
      }

      disconnectedCallback() {
        this.removeEventListeners();
      }

      setProperties() {
        this.productId = this.dataset.productId;
        this.button = this.querySelector('[data-add-to-cart]');
      }

      setEventListeners() {
        this.handleAddToCart = this.addToCart.bind(this);
        this.button?.addEventListener('click', this.handleAddToCart);
      }

      removeEventListeners() {
        this.button?.removeEventListener('click', this.handleAddToCart);
      }

      async addToCart() {
        try {
          const response = await fetch('/cart/add.js', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              id: this.productId,
              quantity: 1
            })
          });
          // Handle response
        } catch (error) {
          console.error('Error adding to cart:', error);
        }
      }
    }

    customElements.define('product-card', ProductCard);
  }
})()

```

## Instructions

1. **Separate JS files** - one file per section in `assets/` directory
2. **Use `defer`** when including scripts
3. **Use `const` and `let`** - never `var`
4. **Use custom elements** to encapsulate logic — guard with `if (!customElements.get('tag-name'))`, use `setProperties()` / `setEventListeners()` / `removeEventListeners()` in `connectedCallback` / `disconnectedCallback`
5. **Pass data via `data-*` attributes**
6. **Select elements only via data attributes** - never by class (e.g. `[data-trigger]`, not `.product-card__button`)
7. **Avoid global scope pollution**
8. **Use observers only when explicitly requested**
9. **Clean up event listeners** in `disconnectedCallback`
