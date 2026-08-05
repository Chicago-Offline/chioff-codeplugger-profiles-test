# chioff-codeplugger-profiles

Public reference profiles and cross-repository integration fixtures for
[`Chicago-Offline/codeplugger`](https://github.com/Chicago-Offline/codeplugger).

Profiles describe channel selection and ordering for a target radio. They do
not contain RF facts, private identities, credentials, radio exports, or
generated codeplugs.

## Profiles

- `profiles/baofeng_dm32/reference.yml` selects one synthetic channel from
  `chioff-ssrf-private` and NOAA WX1 from `ssrf-lite` into a single ordered
  DM-32 zone.

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
