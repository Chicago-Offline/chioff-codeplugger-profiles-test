# chioff-codeplugger-profiles

Public reference profiles and cross-repository integration fixtures for
[`Chicago-Offline/codeplugger`](https://github.com/Chicago-Offline/codeplugger).

Profiles describe channel selection and ordering for a target radio. They do
not contain RF facts, private identities, credentials, radio exports, or
generated codeplugs.

## Profiles

| Fixture | Radio | Profile id | Notes |
| --- | --- | --- | --- |
| `profiles/baofeng_dm32/reference.yml` | `baofeng_dm32` | `chioff_dm32_reference` | Resolves `asg_fixture_reference_simplex` and `asg_wx1` into one `Reference` zone. Generates qdmr YAML in CI. |
| `profiles/baofeng_uv5r_mini/reference.yml` | `baofeng_uv5r_mini` | `chioff_uv5r_mini_reference` | Same two assignments, FM only (this radio is FM-only). Generates CHIRP CSV in CI. |
| `profiles/baofeng_uv5r_mini/negative-dmr.yml` | `baofeng_uv5r_mini` | `chioff_uv5r_mini_negative_dmr` | Negative fixture: resolves a DMR assignment (`asg_dmr_smpx_446500` from `ssrf-lite`) against the FM-only UV-5R Mini. Must fail validation with an explicit unsupported-mode error; CI asserts the failure. |

All assignment IDs resolve from `ssrf-lite` plus the `chioff-ssrf-test`
reference overlay — no real or personal data.

Every profile is validated by CI on push and pull request. A new
profile that CI does not validate is worse than no profile, so add a matching
step in `.github/workflows/validate.yml` in the same change.

## Validate

With sibling checkouts of the three repositories:

```bash
../codeplugger/.venv/bin/codeplugger-profile \
  profiles/baofeng_dm32/reference.yml \
  --ssrf-root ../ssrf-lite/ssrf \
  --ssrf-root ../chioff-ssrf-test/ssrf \
  --radio-root ../codeplugger/radios
```

Generate the disposable qdmr YAML used by the DM-32 headless path with the
same roots and precedence:

```bash
../codeplugger/.venv/bin/codeplugger-profile \
  profiles/baofeng_dm32/reference.yml \
  --ssrf-root ../ssrf-lite/ssrf \
  --ssrf-root ../chioff-ssrf-test/ssrf \
  --radio-root ../codeplugger/radios \
  --output-format qdmr-yaml \
  > dm32-reference.yaml
```

CI validates both profile resolution and qdmr YAML generation. The generated
file is retained briefly as a workflow artifact for inspection; it is not a
source file and must not be committed.

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
