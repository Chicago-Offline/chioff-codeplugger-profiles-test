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
| `profiles/retevis_matetalk_p4/blue.yml` | Retevis MateTalk P4 (`retevis_matetalk_p4`) | `chioff_p4_blue` | 1 (`ChiOff channels`) | Shared ChiOff GMRS F1-F6 and five family DMR assignments | `chioff-ssrf-shared` | Channel layout for `p4_01` / `CO-P4-682041` |
| `profiles/retevis_matetalk_p4/yellow.yml` | Retevis MateTalk P4 (`retevis_matetalk_p4`) | `chioff_p4_yellow` | 1 (`ChiOff channels`) | Shared ChiOff GMRS F1-F6 and five family DMR assignments | `chioff-ssrf-shared` | Channel layout for `p4_02` / `CO-P4-682042` |

The repository also contains `instances.yml`, a registry for the four labeled
P4 radios. It records physical labels and case serials; the yellow radio now
selects the standard P4 profile in this repository.

Each P4's six-digit label number is also its DMR radio ID. The registry stores
that value as `dmr_id` and uses the same number as `dmr_contact_name`, so future
codeplug generation can add every radio to the contact list and identify a
radio by its own number.

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

Validate the P4 profiles with the shared ChiOff overlay:

```bash
for profile in profiles/retevis_matetalk_p4/*.yml; do
  ../codeplugger/.venv/bin/codeplugger-profile "$profile" \
    --ssrf-root ../ssrf-lite/ssrf \
    --ssrf-root ../chioff-ssrf-shared/ssrf \
    --radio-root ../codeplugger/radios \
    --instance-registry instances.yml
done
```

Profiles use explicit SSRF assignment IDs. Assignment objects may provide a
profile-local `display_name` when the same RF channel needs a different label;
the underlying frequency, tone, and offset remain sourced from SSRF.

`--ssrf-root` precedence is positional, so keep the order above consistent with
`.github/workflows/validate.yml` — reordering the roots can change which overlay
wins for a given assignment ID
(see [`codeplugger#5`](https://github.com/Chicago-Offline/codeplugger/issues/5)).

## Adding a radio

`codeplugger` resolves `radio:` against `--radio-root`, so a profile can only
target a radio that ships a `radios/<id>/capabilities.json`. The P4 profiles
require the `retevis_matetalk_p4` radio definition in codeplugger and an
appropriate update to the pinned `ref` in `.github/workflows/validate.yml`.
