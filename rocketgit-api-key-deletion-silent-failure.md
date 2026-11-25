# Security Advisory: Silent Failure of API Key Revocation in RocketGit (< 0.84)

- **Product**: RocketGit (`https://rocketgit.com/user/catalinux/rocketgit`)
- **Vendor**: Catalin(ux) M. BOIE
- **Affected Versions**: `v0.55` through `v0.83` (introduced in commit `e503666df79ef1553d0bb7ffd1d12a6b62748b1c`)
- **Patched Version**: `v0.84`
- **CWE**: CWE-672 (Operation on a Resource after Expiration or Release)
- **CVSS v3.1**: 9.1 (Critical) - `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N`
- **Finder / Credit**: lingyicute (`lingyicute@gmail.com`)

## Summary
In RocketGit versions `0.55` through `0.83`, API key deletion via the web UI silently fails to revoke keys while displaying a success confirmation to the user:
1. `rg_ak_add()` merges new API key rows into the cache without `key_id`, and `rg_ak_valid()` explicitly calls `unset($row['key_id'])` before caching rows.
2. When `rg_ak_list()` renders the settings page from cached rows, the HTML template `user/settings/apikeys/list/line.html` outputs `name="key_delete_ids[]"` with an empty `key_id`.
3. `rg_ak_remove()` formats the incoming key via `sprintf('%u', $key_id)` (producing `0`) and executes `DELETE FROM apikeys WHERE uid = @@uid@@ AND key_id IN (0)` without checking affected rows (`RETURNING key_id`), returning `TRUE` and showing the `delete_ok.html` success message even though zero keys were deleted.

## Impact
Users attempting to revoke compromised or rotated API keys are falsely informed that the deletion succeeded, while the API keys remain active and usable for authenticating API requests.

## Patch / References
- Fix Commit 1 (Cache `key_id`): https://rocketgit.com/user/catalinux/rocketgit/source/commit/9617c54bda655cf93b21546f1fc7030b2e1dc5ac
- Fix Commit 2 (Validate `DELETE ... RETURNING key_id`): https://rocketgit.com/user/catalinux/rocketgit/source/commit/2e40a64ad25e8b2cddd52ccd5d69f10f6b9a6d2a
- Release 0.84: https://rocketgit.com/user/catalinux/rocketgit/source/commit/1242f8367c8ebd33de2d2884b495140e95c864c8
