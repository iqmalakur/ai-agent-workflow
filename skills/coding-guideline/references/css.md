# CSS Formatting Examples

Read [common rules](common.md) and the host-language reference for embedded CSS.
Use the project's CSS formatter, naming, and stylesheet organization.
The shared [HTML example](html.md) provides the registration form fixture.

## Production stylesheet

The stylesheet groups form layout, fields, feedback, controls, focus states, and responsive adjustments.
The local `--accent` property shares the component's accent value.
Both layouts preserve selectors, declaration order, and behavior.

### Incorrect

Rules and declaration groups have no empty-line separation.
A small rule occupies one line among larger multiline rules.

```css
.registration-form {
  --accent: #075985;
  display: grid;
  gap: 1.5rem;
  max-width: 48rem;
  margin-inline: auto;
  padding: 1.5rem;
  color: #17212b;
  font-family: system-ui, sans-serif;
}
.form-grid {
  display: grid;
  gap: 1rem;
}
.field {
  display: grid;
  gap: 0.5rem;
}
.field input {
  box-sizing: border-box;
  width: 100%;
  min-height: 2.75rem;
  padding: 0.625rem;
  border: 1px solid #64748b;
  border-radius: 0.25rem;
  background: #fff;
  font: inherit;
}
.session-options {
  display: grid;
  gap: 0.75rem;
  margin: 0;
  padding: 1rem;
  border: 1px solid #64748b;
}
.form-help,
.form-error {
  margin: 0;
  font-size: 0.875rem;
}
.form-error { color: #b91c1c; }
.field input[aria-invalid="true"] {
  border-color: #b91c1c;
}
.form-actions {
  display: flex;
}
.registration-form button {
  padding: 0.75rem 1.25rem;
  border: 0;
  border-radius: 0.25rem;
  background: var(--accent);
  color: #fff;
  font: inherit;
  cursor: pointer;
}
.registration-form input:focus-visible,
.registration-form button:focus-visible {
  outline: 3px solid var(--accent);
  outline-offset: 3px;
}
.registration-form button:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}
@media (min-width: 40rem) {
  .form-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
  .form-actions {
    justify-content: flex-end;
  }
}
```

### Correct

Separate logical rules and group related declarations within longer rules.
Preserve cascade and fallback order while adjusting layout.
Follow existing project ordering conventions when they are configured.

```css
.registration-form {
  --accent: #075985;

  display: grid;
  gap: 1.5rem;

  max-width: 48rem;
  margin-inline: auto;
  padding: 1.5rem;

  color: #17212b;
  font-family: system-ui, sans-serif;
}

.form-grid {
  display: grid;
  gap: 1rem;
}

.field {
  display: grid;
  gap: 0.5rem;
}

.field input {
  box-sizing: border-box;
  width: 100%;
  min-height: 2.75rem;
  padding: 0.625rem;

  border: 1px solid #64748b;
  border-radius: 0.25rem;
  background: #fff;

  font: inherit;
}

.session-options {
  display: grid;
  gap: 0.75rem;

  margin: 0;
  padding: 1rem;
  border: 1px solid #64748b;
}

.form-help,
.form-error {
  margin: 0;
  font-size: 0.875rem;
}

.form-error {
  color: #b91c1c;
}

.field input[aria-invalid="true"] {
  border-color: #b91c1c;
}

.form-actions {
  display: flex;
}

.registration-form button {
  padding: 0.75rem 1.25rem;

  border: 0;
  border-radius: 0.25rem;
  background: var(--accent);
  color: #fff;

  font: inherit;
  cursor: pointer;
}

.registration-form input:focus-visible,
.registration-form button:focus-visible {
  outline: 3px solid var(--accent);
  outline-offset: 3px;
}

.registration-form button:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

@media (min-width: 40rem) {
  .form-grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .form-actions {
    justify-content: flex-end;
  }
}
```

## Reuse and verification

Reuse existing classes and custom properties when their meaning and scope match the component.
Keep component-specific styles with that component.
Apply function length limits to executable functions in host languages.
Use rule grouping and file responsibility when reviewing stylesheet size.

With the matching HTML fixture, compare relevant computed styles at mobile and desktop widths.
Inspect normal, keyboard-focused, invalid, and disabled states.
Check that rule reordering or extraction preserves specificity, fallback values, and media-query behavior.
Use the project's existing browser test helpers for repeated setup and cleanup.
