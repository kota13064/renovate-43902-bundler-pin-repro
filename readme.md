# Minimal reproduction: Bundler `rangeStrategy=pin` fails with "Value is not updated"

Reproduction for [renovatebot/renovate#43902](https://github.com/renovatebot/renovate/discussions/43902).

## Setup

- [`Gemfile`](Gemfile) with a single ranged dependency: `gem 'rack', '~> 2.2'`
- [`Gemfile.lock`](Gemfile.lock) locking `rack (2.2.23)`
- [`renovate.json`](renovate.json) with `"rangeStrategy": "pin"`

## Current behavior

Renovate generates a `pin` update (`newValue: "2.2.23"`, no quotes), but the auto-replace
step rewrites the line to invalid Ruby by replacing the whole quoted token:

```ruby
gem 'rack', 2.2.23   # quotes dropped
```

Re-extraction of the edited file then fails to find the new value (the bundler regex
requires quotes around `currentValue`), so the edit is rejected and reverted. The debug
log shows:

```
packageFile=Gemfile depName=rack msg="Found match at index ..."
depName=rack manager=bundler packageFile=Gemfile expectedValue=2.2.23 msg="Value is not updated"
branch=renovate/pin-dependencies msg="Error updating branch: update failure"
```

No pin PR is ever created; the branch fails on every run.

## Expected behavior

Renovate opens a pin PR that rewrites the `Gemfile` with a valid, quoted exact version:

```ruby
gem 'rack', '2.2.23'
```

## Notes

- `replace`-type updates are unaffected because ruby versioning's `getNewValue` re-adds
  the quote delimiters, while pin updates use `getPinnedValue` (a bare version string).
- Workaround: set `rangeStrategy: update-lockfile` for the bundler manager, which leaves
  the `Gemfile` untouched.
