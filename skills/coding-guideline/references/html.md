# HTML Formatting Examples

Read [common rules](common.md) and the host-language reference for generated or embedded HTML.
Use the project's HTML formatter and conventions.
Read [CSS](css.md) when changing the associated styles.

## Production markup

The example is an event registration form with contact fields, session choices, guidance, and a submission button.
Its form action is an illustrative local endpoint.
The [CSS example](css.md) supplies styles for these class names.
Both layouts preserve names, attributes, text, and control relationships.

### Incorrect

The form spans multiple lines without separating logical sections.
The email input hides several attributes on a long line.

```html
<form class="registration-form" action="/events/register" method="post">
  <header>
    <h1>Register for the design workshop</h1>
    <p>Choose a session and enter your contact details.</p>
  </header>
  <div class="form-grid">
    <div class="field">
      <label for="attendee-name">Full name</label>
      <input id="attendee-name" name="name" autocomplete="name" required>
    </div>
    <div class="field">
      <label for="attendee-email">Email address</label>
      <input id="attendee-email" name="email" type="email" autocomplete="email" aria-describedby="email-help email-error" aria-invalid="false" required>
      <p id="email-help" class="form-help">We will send your ticket here.</p>
      <p id="email-error" class="form-error" hidden>Enter a valid email.</p>
    </div>
  </div>
  <fieldset class="session-options" aria-describedby="session-help">
    <legend>Workshop session</legend>
    <p id="session-help">Choose <code>AM</code> or <code>PM</code>.</p>
    <label>
      <input type="radio" name="session" value="AM" required>
      Morning, 09:00 to 12:00
    </label>
    <label>
      <input type="radio" name="session" value="PM">
      Afternoon, 13:00 to 16:00
    </label>
  </fieldset>
  <div class="form-actions">
    <button type="submit">Reserve my place</button>
  </div>
</form>
```

### Correct

Separate the introduction, contact fields, session choices, and submission controls.
Keep short elements on one line and wrap the long attribute list.
Preserve the spaces around inline `code` elements in the session instructions.

```html
<form class="registration-form" action="/events/register" method="post">
  <header>
    <h1>Register for the design workshop</h1>
    <p>Choose a session and enter your contact details.</p>
  </header>

  <div class="form-grid">
    <div class="field">
      <label for="attendee-name">Full name</label>
      <input id="attendee-name" name="name" autocomplete="name" required>
    </div>

    <div class="field">
      <label for="attendee-email">Email address</label>
      <input
        id="attendee-email"
        name="email"
        type="email"
        autocomplete="email"
        aria-describedby="email-help email-error"
        aria-invalid="false"
        required
      >
      <p id="email-help" class="form-help">We will send your ticket here.</p>
      <p id="email-error" class="form-error" hidden>Enter a valid email.</p>
    </div>
  </div>

  <fieldset class="session-options" aria-describedby="session-help">
    <legend>Workshop session</legend>
    <p id="session-help">Choose <code>AM</code> or <code>PM</code>.</p>
    <label>
      <input type="radio" name="session" value="AM" required>
      Morning, 09:00 to 12:00
    </label>
    <label>
      <input type="radio" name="session" value="PM">
      Afternoon, 13:00 to 16:00
    </label>
  </fieldset>

  <div class="form-actions">
    <button type="submit">Reserve my place</button>
  </div>
</form>
```

## Reuse and verification

Extract repeated markup through the project's existing component or template system when one is used.
Keep plain HTML examples in their existing document structure.

In a local browser fixture, compare rendered text, form field names, and label associations across both versions.
Check required fields, email validity, radio grouping, and hidden validation feedback.
For validation feedback, update `aria-invalid` and the matching message together.
Also compare inline text spacing and any whitespace-sensitive content.
Use the matching CSS fixture to inspect focused, invalid, disabled, and responsive states.
Browser fixture tests receive the common test size exemptions and reuse requirements.
