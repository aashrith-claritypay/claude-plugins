# claude-plugins

ClarityPay's shared Claude Code plugin marketplace.

## Plugins

- **clarity-review** — baseline PR review policy (severity calibration,
  verification bar, inline-comment output). Repos keep their own
  `REVIEW.md`/`CLAUDE.md` for repo-specific overrides on top of this
  baseline.

## Using this marketplace from a GitHub Actions workflow

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    plugin_marketplaces: "https://github.com/Clarity-Pay/claude-plugins.git"
    plugins: "clarity-review@clarity-plugins"
```

The `clarity-review` skill auto-invokes on review-shaped requests (e.g.
`@claude review this`), or can be invoked explicitly with
`@claude /clarity-review:review`.
