# The Uber TV catalog

The apps Uber TV offers under **Settings → Apps → Get apps**, signed
with the catalog's key. It holds **Uber TV's own apps only** and takes no
submissions: nobody here reviews, or answers for, apps made by others. Those
will come from sources of their own, each signed with its maker's key, that a
user adds to the guide themselves. The guide downloads `catalog.json` and
`catalog.json.sig`, checks the signature against the key it ships with
(`apps/catalog-signers` in the guide), and refuses a catalog older than one it
has already seen. Each app's package is checked against the SHA-256 listed
here before it's unpacked, and its `app.json` must match its listing.

## An app

A folder in `apps/` with an `app.json`. For now every app is a **launcher**: a
website the guide opens full screen in the user's browser (its app window, no
browser bar), with no code run on the computer.

```json
{
  "id": "tv.uber.netflix",
  "name": "Netflix",
  "version": "1.0.0",
  "api": 1,
  "author": "Uber TV",
  "kind": "launcher",
  "description": "Movies and series. Opens Netflix's website full screen; you sign in there.",
  "color": "#e50914",
  "icon": "icon.png",
  "launch": { "web": "https://www.netflix.com/browse" },
  "permissions": { "web": ["netflix.com"] }
}
```

- `id`: reverse-domain, lowercase (`tv.uber.netflix`). `version`: `1.0.0`.
- `launch.web`: a plain https link (no quotes, spaces or `$`), on one of the
  websites in `permissions.web`, lowercase, with no port and no name or
  password in it.
- `icon` (optional): a `.png`, `.jpg` or `.webp` in the folder, 512 KB at most.
- A package holds only `app.json`, pictures, `.md` and `.txt` files and a
  plain-text `LICENSE` or `COPYING`: 200 files, 400 files and folders, four
  folders deep and 20 MB at most, no links, no programs.

## Building and publishing

```bash
./build                      # packs apps/ into packages/, writes and signs catalog.json
git add -A && git commit -m "…" && git push
```

`build` uses the guide's own `bin/om-pg-apps pack`, so an app that wouldn't
install won't pack, and it reads the signed catalog back with the guide's keys
before it's done. Signing needs a private key: `~/.ssh/uber-tv-catalog`, with
a passphrase, never in this repo. Keep a copy of it somewhere safe (it's
encrypted with the passphrase, so a password manager or a USB stick will do).

**The keys.** The guide trusts two (`apps/catalog-signers` in the guide, public
halves only): the one above, and a spare (`uber-tv-catalog-spare`) kept
offline.

- **Lost the key:** sign with the spare (`./build --key <the spare>`). Guides
  already trust it; nothing else changes. Then make a new spare and add its
  public half to the guide.
- **Key stolen:** sign with the spare, and take the stolen key's line out of
  the guide's `apps/catalog-signers` in its next release. A guide with that
  release installs again, from the catalog signed with the spare, any app it
  had installed under the stolen key. Until people update, the thief could
  still sign catalogs their guides accept, so release quickly.

**Updating an app:** change it, raise its `version`, build. A guide that has it
installs the update by itself if it asks for nothing new; if it asks for more,
it waits until the user agrees.

**Pulling a bad app:** add it to `revoked.json` and build:

```json
[{ "id": "tv.uber.example", "version": "1.0.0", "reason": "It sent people to the wrong site." }]
```

(`"version": "*"` pulls every version.) A guide that has it turns it off and
tells the user why, and a pulled version is never offered or installed, even
if its folder is still in `apps/` (it's still packed and listed). A catalog
whose `revoked` the guide can't read is refused whole.
