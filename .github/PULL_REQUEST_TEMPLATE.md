## What and why

<!-- What does this change, and why? Link the issue it closes (Fixes #123). -->

## Checklist

- [ ] The title follows [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `docs:`, …).
- [ ] Nothing existing breaks: the SDK stays on 1.0.x and only adds to its public API.
- [ ] Tests cover the change, and `npm run ci` passes locally.
- [ ] Public API changes are reflected in `review/stowcards-sdk.api.md` (`npm run api:update`) and the README.
- [ ] Commits are signed off (`git commit -s`) under the [Developer Certificate of Origin](https://developercertificate.org/).
