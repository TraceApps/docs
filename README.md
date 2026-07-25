# TraceApps Documentation

Source for **https://traceapps.github.io/docs/**. Covers the TraceApps self-hosted family: [CookTrace](https://github.com/TraceApps/cooktrace), [LiftTrace](https://github.com/TraceApps/lifttrace), [NutriTrace](https://github.com/TraceApps/nutritrace).

Built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and deployed to GitHub Pages via the `deploy.yml` workflow.

## Local preview

```bash
pip install -r requirements.txt
mkdocs serve
```

Then open http://127.0.0.1:8000.

## Contributing

Every page in `docs/` is plain markdown. To fix a typo or add a page:

1. Fork this repo, or click the pencil icon at the top of any published page to edit in-browser.
2. Open a PR against `main`.
3. Merged PRs auto-deploy on push.

See the [Contribute → Translations](https://traceapps.github.io/docs/contribute/translations/) page for how to contribute translations to the apps themselves (translations live in each app's repo, not here).

## License

Documentation is [AGPL-3.0](LICENSE), matching the three apps it covers.
