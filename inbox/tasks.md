# Tasks Inbox

Tasks pending execution. The `task-processor` skill picks the top-most task and works down the list.

## Format

```
- [ ] [TASK-ID] Description of the task
  Spec: path/to/spec.md
  Notes: Any relevant context
```

---


- [x] [TASK-001] Replace Tailwind CDN with production build
  Notes: Currently loading Tailwind via `cdn.tailwindcss.com` play script in `templates/structure/header.gohtml`. This compiles CSS in the browser at runtime — not suitable for production. Replace with Tailwind CLI standalone binary (no Node.js needed). Steps: create `tailwind.config.js` with current config (primary color `#615FFF`, Roboto font), create source CSS with `@tailwind` directives, build to `assets/css/tailwind.css`, replace CDN `<script>` with `<link>` to built CSS, add build step to `Makefile`. Scan `templates/**/*.gohtml` and `assets/js/**/*.js` for class detection.
