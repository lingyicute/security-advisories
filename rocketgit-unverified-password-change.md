# Security Advisory: Unverified Password Change in RocketGit (< 0.84)

- **Product**: RocketGit (`https://rocketgit.com/user/catalinux/rocketgit`)
- **Vendor**: Catalin(ux) M. BOIE
- **Affected Versions**: `v0.40` through `v0.83` (introduced in commit `e9197291380929dd53f02f7373e8a339d6bd65a2`)
- **Patched Version**: `v0.84`
- **CWE**: CWE-620 (Unverified Password Change)
- **CVSS v3.1**: 8.8 (High) - `CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`
- **Finder / Credit**: lingyicute (`lingyicute@gmail.com`)

## Summary
In RocketGit versions `0.40` through `0.83`, the password verification helper `rg_user_pass_valid()` in `inc/user.inc.php` calculates the verification result in `$ret` (initializing `$ret = FALSE` and breaking out of the validation loop when the password is empty or does not match the stored hash), but unconditionally executes `return TRUE;` at line 1006 instead of `return $ret;`.

## Impact
When changing a user's password in `inc/user/pass/pass.php`, the check `if (!rg_user_pass_valid($db, $ui_login['uid'], $old_pass))` always succeeds regardless of the supplied `old_pass` value. Anyone with temporary access to an authenticated session can permanently change the account password using a completely fabricated old password, resulting in full account takeover.

## Patch / References
- Fix Commit: https://rocketgit.com/user/catalinux/rocketgit/source/commit/6dd5b260a1b844335f49daf842c6584b6f1ae279
- Release 0.84: https://rocketgit.com/user/catalinux/rocketgit/source/commit/1242f8367c8ebd33de2d2884b495140e95c864c8
