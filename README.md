# chioff-codeplugger-profiles

Public reference profiles and cross-repository integration fixtures for
[`Chicago-Offline/codeplugger`](https://github.com/Chicago-Offline/codeplugger).

Profiles describe channel selection and ordering for a target radio. They do
not contain RF facts, private identities, credentials, radio exports, or
generated codeplugs.

## Profiles

| Profile | Radio | Profile `id` | Zones | Assignments | SSRF roots | Purpose |
|---|---|---|---|---|---|---|
| `profiles/baofeng_dm32/reference.yml` | Baofeng DM-32 (`baofeng_dm32`) | `chioff_dm32_reference` | 1 (`Reference`) | `asg_fixture_reference_simplex`, `asg_wx1` | `chioff-ssrf-private`, `ssrf-lite` | Minimal end-to-end fixture: explicit assignment IDs resolving across two overlays into one ordered zone |

These are **synthetic fixtures, not a radio fleet.** Nothing here maps to a
physical radio, and no profile should acquire an owner, tape color, case serial,
or installed-codeplug record — that class of per-device fact belongs in a
private instance registry, not a public test repository.

Every profile in the table is validated by CI on push and pull request. A new
profile that CI does not validate is worse than no profile, so add a matching
step in `.github/workflows/validate.yml` in the same change.

## Validate

With sibling checkouts of the three repositories:

```bash
../codeplugger/.venv/bin/codeplugger-profile \
  profiles/baofeng_dm32/reference.yml \
  --ssrf-root ../ssrf-lite/ssrf \
  --ssrf-root ../chioff-ssrf-private/ssrf \
  --radio-root ../codeplugger/radios
```

The reference profile uses explicit SSRF assignment IDs. Changes to RF data or
display names belong in an SSRF overlay rather than this repository.

`--ssrf-root` precedence is positional, so keep the order above consistent with
`.github/workflows/validate.yml` — reordering the roots can change which overlay
wins for a given assignment ID
(see [`codeplugger#5`](https://github.com/Chicago-Offline/codeplugger/issues/5)).

## Adding a radio

`codeplugger` resolves `radio:` against `--radio-root`, so a profile can only
target a radio that ships a `radios/<id>/capabilities.json`. At the ref this
repository pins, that is `baofeng_dm32` only. Adding a profile for any other
radio requires the radio definition to land in `codeplugger` first, then a
deliberate bump of the pinned `ref` in `.github/workflows/validate.yml`.
