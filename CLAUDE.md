@../APP_SCAFFOLD.md

## Testing

Ship tests **with** every change — part of "done", not a follow-up:

- **Unit tests** in `<App>Tests/` for all new logic (models, generators, managers, scoring, persistence); cover edge cases and seeded-RNG determinism, and assert every generated level is completable.
- **UI tests** in `<App>UITests/` for any new or changed screen/flow — at minimum a smoke test that it navigates without crashing.
- Fixing a bug? Add a test that fails without the fix. Keep the suite green.

See the full **Testing Policy** in `../APP_SCAFFOLD.md` (Section 10).
