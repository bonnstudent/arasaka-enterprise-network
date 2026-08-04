# CIFS/Kerberos Mount Failure — cifs-utils 7.6 cruid Regression

**Environment:** AlmaLinux 9.8 client joined to a Windows Server AD domain, CIFS shares on Windows file servers

## Symptom

Domain users on the AlmaLinux client could not mount their Kerberos-authenticated CIFS
home shares. Mounts either failed outright or authenticated as the wrong principal —
every user's mount resolved to root's Kerberos ticket rather than their own.

## What I checked

1. **Kerberos ticket** — `klist` confirmed the user had a valid TGT after login.
2. **Domain join and DNS** — the client resolved the domain and could reach the DCs.
3. **Share permissions** — NTFS and share-level permissions were correct for the user.
4. **Mount options** — the `cruid=` option was present, which should tell `cifs.upcall`
   which user's credential cache to use.

The configuration was correct, so the problem was in how the credential cache was being
located rather than in the mount itself.

## Root cause

Two issues compounded:

1. `cifs-utils` 7.6 has a regression where `cifs.upcall` ignores the `cruid=` mount
   option and falls back to root's credential cache.
2. The system default credential cache was `KEYRING`, which is not reliably accessible
   to the upcall helper running in a different context.

## Fix

Changed the default credential cache in `/etc/krb5.conf` from KEYRING to a file-based
cache:

    default_ccache_name = FILE:/tmp/krb5cc_%{uid}

With a file-based cache, `cifs.upcall` could locate the correct per-user ticket, and an
explicit `kinit` on login populated it.

## Validation

- Logged in as separate domain users and confirmed each mounted their own home share
- Verified with `klist` that the correct principal was in use per session
- Confirmed read/write access matched the user's assigned permissions

## Takeaway

Correct configuration does not guarantee correct behaviour — a regression in a package
version can invalidate documented options. Checking the package version and its known
issues is worth doing before assuming the configuration is at fault.
