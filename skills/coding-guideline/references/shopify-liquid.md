# Shopify Liquid Formatting Examples

Read [common rules](common.md) and [HTML](html.md) when changing template markup.
Add [CSS](css.md) or [JavaScript](javascript-typescript.md) guidance when those embedded languages change.
Use the project's Liquid formatter and Shopify Theme Check configuration.

## Production template

The collection grid displays a heading, product count, cards, availability, and an empty state.
Shopify supplies `collection`, product objects, and the `escape` and `money` filters.
Both versions preserve the product URL, title, vendor, price, and availability text.

### Incorrect

The grid lacks empty lines between logical groups.
The availability branches duplicate the full product-card markup.

```liquid
<section class="collection-grid">
  <header>
    <h2>{{ collection.title | escape }}</h2>
    <p>{{ collection.products_count }} products</p>
  </header>
  {% if collection.products.size > 0 %}
    <ul class="products">
      {% for product in collection.products %}
        <li>
          {% if product.available %}
          <article class="product-card">
            <h3><a href="{{ product.url | escape }}">{{ product.title | escape }}</a></h3>
            <p>{{ product.vendor | escape }}</p>
            <p>{{ product.price | money }}</p>
            <p class="availability">Available</p>
          </article>
          {% else %}
          <article class="product-card">
            <h3><a href="{{ product.url | escape }}">{{ product.title | escape }}</a></h3>
            <p>{{ product.vendor | escape }}</p>
            <p>{{ product.price | money }}</p>
            <p class="availability">Sold out</p>
          </article>
          {% endif %}
        </li>
      {% endfor %}
    </ul>
  {% else %}
    <p>No products are available in this collection.</p>
  {% endif %}
</section>
```

### Correct

Keep the collection structure in the section and reuse the card through `render`.
Pass the product explicitly to preserve the snippet's data dependency.

The section markup lives in `sections/collection-grid.liquid`.

```liquid
<section class="collection-grid">
  <header>
    <h2>{{ collection.title | escape }}</h2>
    <p>{{ collection.products_count }} products</p>
  </header>

  {% if collection.products.size > 0 %}
    <ul class="products">
      {% for product in collection.products %}
        <li>
          {% render 'product-card', product: product %}
        </li>
      {% endfor %}
    </ul>
  {% else %}
    <p>No products are available in this collection.</p>
  {% endif %}
</section>
```

The reusable card lives in `snippets/product-card.liquid`.

```liquid
<article class="product-card">
  <h3>
    <a href="{{ product.url | escape }}">{{ product.title | escape }}</a>
  </h3>
  <p>{{ product.vendor | escape }}</p>
  <p>{{ product.price | money }}</p>

  {% if product.available %}
    <p class="availability">Available</p>
  {% else %}
    <p class="availability">Sold out</p>
  {% endif %}
</article>
```

## Whitespace, schema, and size

Preserve spaces inside text and between inline elements when changing tag layout.
Use `{%-` and `-%}` only when removing their surrounding whitespace preserves the rendered content.
Apply the shared file limits when reviewing section and snippet responsibility.
Keep a section's `{% schema %}` JSON with its owning section while extracting reusable markup.
Review Liquid tag blocks through logical grouping.
Apply function limits to actual embedded executable functions.

## Verification

Render both versions with populated, unavailable, and empty collection fixtures.
Include a product title containing `&` or `<` to check escaping.
Compare visible text, links, prices, and card order after extraction.
Check that `render` receives every created variable required by the snippet.
Run available Theme Check and formatter tooling.
Identify custom filters or runtime stubs used by local rendering checks.
Use existing shared render fixtures and cleanup helpers when tests repeat these operations.
