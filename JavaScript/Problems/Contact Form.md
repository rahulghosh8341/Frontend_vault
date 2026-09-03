---
title: Building a "Contact Us" form that submits using native HTML
aliases:
  - Contact Form
difficulty: Easy
time: 10 min
languages:
  - HTML
companies:
  - "[[Lyft]]"
  - "[[Amazon]]"
  - "[[OpenAI]]"
  - "[[Stripe]]"
  - "[[PayPal]]"
pattern:
  - "[[HTML Form Submission]]"
concepts:
  - "[[HTML Forms]]"
solved: true
solvedDate: 2026-08-27
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Build a basic contact form using only native HTML form submission — no JavaScript required.

## Problem

Build a "Contact Us" form with:
- **Name** field (single-line text input)
- **Email** field (email input)
- **Message** field (multi-line textarea)
- **Submit** button with text "Send"

On submit, form must POST to `https://questions.greatfrontend.com/api/questions/contact-form` with body fields: `name`, `email`, `message`.

No JavaScript or client-side validation — rely on native HTML form submission.

## Companies

- [[Lyft]]
- [[Amazon]]
- [[OpenAI]]
- [[Stripe]]
- [[PayPal]]

## Pattern

- [[HTML Form Submission]]

## 🤔 Thought Process

Problem explicitly says: "Do not use any JavaScript or framework-specific features" and "form submission should implemented entirely in HTML". This means:
- Use `<form action="..." method="POST">` — browser handles serialization and POST.
- Every input needs a `name` attribute matching the API's expected keys (`name`, `email`, `message`).
- `<textarea>` for multi-line message field.
- Submit button needs `type="submit"`.
- No `onSubmit` handler or `preventDefault()` needed.

## 💻 Final Solution

### Native HTML (Required)

```html
<form action="https://questions.greatfrontend.com/api/questions/contact-form" method="POST">
  <div>
    <label for="name">Name</label>
    <input type="text" id="name" name="name" required />
  </div>
  <div>
    <label for="email">Email</label>
    <input type="email" id="email" name="email" required />
  </div>
  <div>
    <label for="message">Message</label>
    <textarea id="message" name="message" required></textarea>
  </div>
  <button type="submit">Send</button>
</form>
```

### React/JSX Version (for reference only — problem requires plain HTML)

```jsx
import submitForm from './submitForm';

export default function App() {
  return (
    <form action="https://questions.greatfrontend.com/api/questions/contact-form" method="POST" onSubmit={submitForm}>
      <input type="text" name="name" />
      <input type="email" name="email" />
      <textarea name="message"></textarea>
      <button type="submit">Send</button>
    </form>
  );
}
```

> **Note:** JSX requires lowercase `<textarea>` (not `<textArea>`). The plain HTML version is the correct answer for this GFE question.

## UI Output

![[contact-form-output.png]]

## 🤔 Why This Works

- `<form action>` + `method="POST"` → browser performs HTTP POST on submit.
- `name` attributes serialize as `application/x-www-form-urlencoded` body: `name=...&email=...&message=...`
- `<textarea>` appropriate for multi-line input.
- Native validation (`required`) provides basic UX without JS; server handles full validation.

## 🐞 Bugs I Made

- Used `<textArea>` (camelCase) in JSX — must be lowercase `<textarea>`.
- Initially tried React `onSubmit` + `submitForm` import — problem forbids JavaScript entirely.

## Production Considerations

- Add `<label for="...">` linked via `id` for accessibility.
- Use `required` attribute for native validation (optional but recommended).
- For file uploads, add `enctype="multipart/form-data"` to `<form>`.
- In SPA/React apps, use `e.preventDefault()` + `fetch()` for AJAX submission instead.

## ⭐ Revision Notes

### Key Facts

- Native form submission: `<form action="URL" method="POST">` — no JS needed.
- `name` attribute is mandatory for form data serialization.
- `<textarea>` for multi-line text; no `value` prop, children = content.
- JSX: all tags lowercase (`textarea`, not `textArea`).

### Common Interview Questions

- How does native form submission work? → Browser serializes named controls, sends POST to `action` URL.
- Why avoid `onSubmit` in plain HTML question? → Problem explicitly forbids JS; native `<form>` handles it.

### Interview Takeaways

- Recognize when native HTML suffices — avoids unnecessary JS.
- Accessibility: labels + ids matter.
- React/JSX has stricter casing rules than HTML.

### Related

- [[HTML Form Submission]]
- [[HTML Forms]]
