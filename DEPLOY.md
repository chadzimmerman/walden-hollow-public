# Publishing this site

Plain HTML and CSS. No build step, no framework, no dependencies. `index.html`
is the site, and opening it in a browser is an accurate preview of production.

## Put it on GitHub Pages

1. Create a **public** repository (this content is meant to be read).
2. Push these files to `main`.
3. Repo → **Settings → Pages** → Source: *Deploy from a branch*, Branch: `main`, folder: `/ (root)`.
4. It appears at `https://<user>.github.io/<repo>/` within a minute or two.

`README.md` shows on the repository page; `index.html` is what Pages serves. The two audiences, an engineer reading the repo and a club board reading the site, each land on the right one.

### A custom domain

Do this **before** the site is sent to anyone, and before the privacy policy and
support URLs go into App Store Connect. Nothing in the markup assumes a path, so
it works without edits.

1. In **Settings > Pages > Custom domain**, enter the hostname. GitHub writes a
   `CNAME` file to the repo root; leave it alone after that.
2. DNS, depending on which form you want as primary:
   - `www.example.com`: one `CNAME` record pointing at `<user>.github.io`.
   - Bare `example.com`: four `A` records at GitHub's apex IPs (and the matching
     `AAAA` records), or an `ALIAS`/`ANAME` if the registrar supports one.
     GitHub's own advice is to make `www` primary and redirect the apex to it.
3. Wait for **Enforce HTTPS** to become tickable, then tick it. It stays greyed
   out until DNS resolves and a Let's Encrypt certificate is issued, usually
   minutes. Do not publish the URL to Apple before this, or the privacy policy
   gets fetched over plain HTTP.

### Changing the domain later

A Pages site serves **one** custom domain at a time. Repointing it at a new
domain does not keep the old one working: the old hostname stops resolving to
this site the moment `CNAME` changes.

By then the old URL exists in places you do not control, including the privacy
policy and support URLs held in App Store Connect and Google Play, and any link
mailed to a club. So when the product name is settled and the domain moves:

- Keep renewing the old domain. About $12/yr.
- Set a **301 redirect** from it to the new one. Registrar URL forwarding or a
  Cloudflare redirect rule both do this for free, and neither needs a server.
- Update the privacy policy and support URLs in both store listings. Those are
  metadata and can be edited without submitting a new build.

Note that the club's own domain may not be a placeholder at all. That club really
is Walden Hollow and will keep running this app, so `waldenhollow.com` can stay
pointed at the club site permanently while the product name gets its own domain.

---

## Before it goes public

Search the files for `TODO`. There are four, and every one is a placeholder
that must not ship:

| Where | What |
|---|---|
| `index.html`, `privacy.html`, `support.html` | `hello@waldenhollow.app`, the real support address |
| `privacy.html` | `[LEGAL ENTITY NAME]` and `[ADDRESS]` |
| `privacy.html` | `[DATE]`, the effective date |

Also confirm the **pricing** in `index.html` is what you actually intend to
charge publicly. It is written as `$5` per member per month, billed to the club,
with setup quoted separately.

```bash
grep -rn "TODO\|\[LEGAL ENTITY\|\[ADDRESS\]\|\[DATE\]" *.html
```

## Screenshots

The phone frames are empty until you add images. They render a labelled
placeholder in the meantime, so an unfinished site reads as unfinished rather
than broken.

In the app repository:

```bash
npm run screenshots
```

That writes six PNGs at 1320x2868. Those full-size files are what App Store
Connect and Google Play want, so keep them.

For the site they get resized and converted, because the raw set is about 6.8 MB
and the map alone is 4.2 MB, which is far too heavy for a landing page. 660px
wide is still sharp at the size the frames render them:

```bash
cd assets/screens
for f in *.png; do
  sips -Z 660 -s format jpeg -s formatOptions 88 "$f" --out "${f%.png}.jpg"
  rm "$f"
done
```

That takes the set to roughly 580 KB, a twelvefold saving with no visible loss.
The markup expects `01-home.jpg`, `02-map.jpg`, `03-log.jpg`, `04-history.jpg`
and `06-fish.jpg`. Nothing else needs changing: the frames pick them up, and each
`<img>` removes itself if the file is absent.

`05-board.jpg` is captured but not currently placed on the page. It is there for
the store listings.

**These must come from sample mode**, which runs on a fictional river. The app's
own test suite enforces that the sample data is nowhere near the club's real
water; do not substitute screenshots taken against live club data.

## What is deliberately not here

- **No analytics, no cookies, no third-party requests.** Fonts are self-hosted
  rather than linked from a font CDN, because a CDN font request reports every
  visitor's IP to that provider, which is a poor look on a site whose argument
  is that a private club's data stays private. It also means the privacy page
  can say "this site makes no third-party requests" and have it be true.
- **No JavaScript**, beyond one inline `onerror` per screenshot that removes a
  missing image. The FAQ accordions are `<details>` elements.
- **No build tooling.** Editing copy means editing an HTML file.
