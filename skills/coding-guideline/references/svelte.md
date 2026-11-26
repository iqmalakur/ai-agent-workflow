# Svelte Coding Guidelines

Read [common rules](common.md) and the applicable references for [script](javascript-typescript.md), [markup](html.md), and [styles](css.md).
Follow the language, aliases, and formatter configured in the target project.

## Component reuse

- Check the project for existing components before creating one. Reuse a component that already meets the requirement.
- Extend an existing component when a small change fits its responsibility.
- Create a new component when reuse or a small extension cannot meet the requirement.

## Changes to shared components

- Before changing a shared component, review every existing use and identify which uses the change would affect.
- Preserve the component API, default behavior, and appearance for existing uses outside the authorized scope.
- For behavior specific to one use, prefer an optional prop that preserves the default behavior.
- Treat a clear user request as approval for its stated scope, such as restyling every button.
- If a necessary change would affect existing uses outside the requested scope, explain the impact and get approval before editing.
- After editing, verify the expected behavior at the affected call sites.

## Component structure

- Use PascalCase for reusable component filenames. Keep SvelteKit route filenames such as `+page.svelte` and `+layout.svelte`.
- Put `<script>` first, markup next, and an optional scoped `<style>` block last.
- In `<script>`, place imports first and props next. Group state, derived values, and handlers by responsibility.
- Use one empty line between logical groups. In longer scripts, follow the surrounding style for section comments.
- Reuse shared stores and helpers. Use existing aliases such as `$components`, `$store`, and `$helpers` where defined.
- The JavaScript example uses two-space indentation and single quotes. It omits semicolons and trailing commas.
- Document the props shape with JSDoc in JavaScript components. Use TypeScript types when the project uses `<script lang="ts">`.

## Reactivity and markup

- In Svelte 5 runes components, destructure `$props()` and put optional defaults in the destructuring pattern.
- Use `$state()` for mutable local state and `$derived()` for computed values. Use `$derived.by()` for multiline computations.
- Use `$effect()` for reactive side effects. Keep the existing syntax in legacy components unless the task includes a migration.
- Use DOM event attributes such as `onclick` and pass component notifications through callback props such as `onSelect`.
- Use `$bindable()` for props that callers need to bind in both directions. Use `bind:this` when the script needs the element reference.
- Render snippet props with `{@render ...}`. Use `{@render children?.()}` when children are optional.
- Use a stable key in `{#each}` when a list can reorder or entries need to retain their identity.
- Use directives such as `class:active` for simple conditional classes. Wrap long attribute lists at attribute boundaries.

## SVG icons

- Prefer a reusable `.svelte` component with inline `<svg>` markup for UI icons, including SVGs supplied by the user.
- Place the component with existing icons. Keep a separate `.svg` file only when another consumer requires it.
- Preserve the `viewBox` and geometry. Follow the existing icon API, such as `size` for dimensions and `color` for `fill` or `stroke`.
- Use the existing theme default for configurable colors. Preserve fixed colors that are part of the artwork.

## Example

`ItemPicker.svelte` keeps the selected ID locally and notifies the parent when a user selects an item.

```svelte
<script>
  /**
   * @type {{
   *   items: { id: string, label: string }[],
   *   onSelect?: (id: string) => void
   * }}
   */
  let { items, onSelect } = $props()

  /** @type {string | null} */
  let selectedId = $state(null)
  let selectedItem = $derived(items.find((item) => item.id === selectedId))

  /** @param {string} id */
  const handleSelect = (id) => {
    selectedId = id
    onSelect?.(id)
  }
</script>

<ul>
  {#each items as item (item.id)}
    <li>
      <button
        type="button"
        class:active={item.id === selectedId}
        aria-pressed={item.id === selectedId}
        onclick={() => handleSelect(item.id)}
      >
        {item.label}
      </button>
    </li>
  {/each}
</ul>

{#if selectedItem}
  <p>Selected: {selectedItem.label}</p>
{/if}

<style>
  .active {
    font-weight: 600;
  }
</style>
```
