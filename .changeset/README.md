# Changesets

This folder manages versioning for the Authifi documentation.

## Adding a Changeset

When making documentation changes that should appear in the changelog, run:

```bash
npm run changeset:add
```

### Version Guidelines

- **patch** (0.0.x): Typo fixes, minor clarifications, formatting changes
- **minor** (0.x.0): New documentation pages, significant content additions, new sections
- **major** (x.0.0): Major restructuring, breaking URL changes, complete rewrites

## Releasing

To consume changesets and update the version:

```bash
npm run version
```

## More Information

See the [Changesets documentation](https://github.com/changesets/changesets) for details.
