---
aliases:
  - HTML Form Submission
---

## Core Idea

Rely on native HTML `<form>` attributes (`action`, `method`, `enctype`, `name`) to serialize user inputs and transmit HTTP requests without requiring client-side JavaScript.

## Recognition

- Problem asks for form submission using native HTML features.
- Request payload keys mapped from `name` attributes of controls.
- Endpoint expects `POST` or `GET` submission via URL encoding or multipart data.

## Template

```html
<form action="https://api.example.com/submit" method="POST">
  <div>
    <label for="username">Username</label>
    <input id="username" type="text" name="username" required />
  </div>
  <div>
    <label for="bio">Bio</label>
    <textarea id="bio" name="bio"></textarea>
  </div>
  <button type="submit">Submit</button>
</form>
```

## Variations

- **Default Encoding:** `application/x-www-form-urlencoded`
- **File Upload Encoding:** `enctype="multipart/form-data"`
- **Client JS Handler:** `onSubmit={handleSubmit}` with `e.preventDefault()` for Single Page Applications (SPA).

## Complexity

- **Time:** $O(N)$ where $N$ is number of form controls serialized by browser.
- **Space:** $O(N)$ for request payload.

## Common Mistakes

- Using JSX camelCase tag names like `<textArea>` (HTML elements must be lowercase in JSX).
- Forgetting `name` attributes (unnamed fields are omitted from form payload).
- Omitting `<label>` elements (hurts accessibility and usability).

## Interview Tips

- Mention accessibility: associate `<label>` with `<input>` using `for`/`htmlFor` + `id`.
- Note difference between native submit (causes page reload/redirect) and AJAX/Fetch submission.

## Problems Using This Pattern

- [[Contact Form]]

## Related Patterns

- None

## Related Concepts

- [[HTML Forms]]
