# luna-doc

## Spell checking

This repo uses [codespell](https://github.com/codespell-project/codespell)
for lightweight typo checks.

Install it locally with one of:

```sh
pipx install codespell
python -m pip install --user codespell
```

Run the configured check from the repository root:

```sh
codespell
```

Skips and project-specific settings live in `.codespellrc`. Generated site
output, binary assets, and data/sample files are excluded.
