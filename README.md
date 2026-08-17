<p align="center">
  <img src="assets/logo.svg" alt="The Leap Second Preservation Society" width="620">
</p>

<p align="center">
  <strong>37 seconds of heritage. 9 years to save them.</strong><br>
  Because the Earth cannot keep time, and should not have to.
</p>

---

This repository contains the public site for The Leap Second Preservation
Society (leapsecondsociety.org, unregistered in every sense), a heritage
trust campaigning to save the leap second from abolition.

## The Society

The Society exists to protect the 27 leap seconds inserted into UTC since
1972, each one added by hand, each one irreplaceable, and none of them
welcome. It maintains the National Register of leap seconds, operates an
adoption programme through which any of the 27 may be adopted for a one-time
contribution of nothing, and runs a petition whose signatures are counted,
totalled, and presented to no one. The Society's position is that the leap
second is a failure of computers, not of seconds, and it will hold that
position until 2035, at which point it will hold it quietly.

## The facts, regrettably

The premise is real. TAI currently leads UTC by exactly 37 seconds: 10
seconds absorbed when UTC was defined in 1972, plus 27 positive leap seconds
inserted between June 1972 and December 2016. And in November 2022 the 27th
General Conference on Weights and Measures (CGPM, Resolution 4) resolved to
discontinue the leap second by or before 2035. The parts of this site that
sound invented are the charity. The parts that sound official are true.

---

## Development notes

The parody ends here. The rest of this file is accurate.

### Layout

A static, zero-build, zero-dependency site. Two HTML files and a handful of
generated images. There is no framework, no bundler and no `package.json`.
Cloudflare Pages serves the repository root exactly as it appears here. The
petition counter, the registry table and the certificate generator are all
inline vanilla JS at the bottom of `index.html`.

```
index.html            the site
404.html              catch-all, served automatically by Cloudflare Pages
favicon.svg           icon source of truth (64px grid, text outlined)
favicon.ico           16/32/48, generated
apple-touch-icon.png  180x180, generated
og.png                1200x630 share image, generated
assets/logo.svg       wordmark, text outlined, used at the top of this README
tools/og.html         source for og.png
tools/logo-src.svg    source for assets/logo.svg, text still live
tools/favicon-16.svg  pixel-grid 16px icon, used for the smallest .ico entry
Makefile              asset regeneration only, never runs at deploy time
_headers              Cloudflare Pages header rules
robots.txt            permissive
wrangler.toml         Cloudflare Pages configuration
```

The page makes zero requests to any external domain. Type is Helvetica Neue
with Arial and generic sans fallbacks, so there are no webfonts to host or
wait for. The design is a conservation-campaign poster: white ground, heavy
grotesk headings, and a single hot orange spent on the endangerment notice,
the calls to action, and the full stop after the 37.

### The production domain

`leapsecondsociety.org` is a placeholder until the domain is actually
purchased. It is hardcoded, deliberately, in three files, and nothing
derives it from anything else:

| File | What to change |
| --- | --- |
| `index.html` | `rel=canonical`, `og:url`, `og:image`, `twitter:image` |
| `404.html` | nothing, the 404 uses only root-relative paths |
| `tools/og.html` | the domain printed in the footer of the share image |
| `README.md` | this table, and the mentions above it |

After changing `tools/og.html`, re-run `make og`.

### Local preview

```sh
make serve          # python3 -m http.server 8000
```

Then open `http://localhost:8000`. A local server is preferable to opening
the file directly because the icon paths are root-absolute. `_headers` is
applied by Cloudflare, not by the local server.

### Regenerating images

Only needed when the tagline, the wordmark or the icon changes. Requires
`google-chrome`, ImageMagick 7 (`magick`) and Inkscape on the machine doing
the regenerating; none of them is needed to deploy, because the outputs are
committed.

```sh
make assets         # everything below
make og             # og.png     <- tools/og.html, via headless Chrome
make favicon        # favicon.ico + apple-touch-icon.png <- the SVG sources
make logo           # assets/logo.svg <- tools/logo-src.svg, text outlined
```

No font gymnastics are required: the campaign face is the Helvetica/Arial
stack, which resolves on this machine to Liberation Sans, metric-compatible
with what most non-Apple visitors see in the browser. `make favicon` needs
no fonts at all; the text in `favicon.svg` and `tools/favicon-16.svg` is
already paths and pixels.

`make logo` outlines the wordmark's text so the README renders the same
regardless of the viewer's fonts. Inkscape rewrites the whole file, so the
`GENERATED` comment at the top has to be pasted back afterwards.

### Deploying

Wrangler is configured via `wrangler.toml`, so a deploy is one command from
an authenticated shell:

```sh
make deploy         # wrangler pages deploy .
```

### Which Cloudflare account this deploys to

This machine has two Cloudflare identities, and picking the wrong one
deploys this site into an unrelated organisation.

**Pages configuration cannot pin the account.** `account_id` is a
Workers-only key; putting it in a Pages `wrangler.toml` makes Wrangler
refuse to run:

```
Configuration file for Pages projects does not support "account_id"
```

So the account is selected by **an auth profile bound to this directory**,
recorded in `~/.config/.wrangler/profiles/directory-bindings.json`:

```sh
wrangler auth activate personal    # re-run after moving or recloning the repo
wrangler whoami                    # must print: Active profile: personal
```

Without a binding, Wrangler falls back to the `default` profile, which here
is the other organisation, and it will deploy there without asking. **Check
`whoami` before deploying.** The binding lives outside the repo, so a fresh
clone, a moved directory, or another machine all need `wrangler auth
activate` again.

One extra trap: Wrangler caches the resolved account in the untracked
`.wrangler/cache/wrangler-account.json` inside this directory. If a deploy
ever went to the wrong account from here, activating the right profile is
**not** enough; delete `.wrangler/` as well, or the cached account ID wins
and the API call fails with `Authentication error [code: 10000]`.

For CI, where profiles do not exist, set `CLOUDFLARE_ACCOUNT_ID` (the
account to deploy into) and `CLOUDFLARE_API_TOKEN` (credentials scoped to
it) as environment variables.

The Pages project is `leapsecondsociety`, production branch `main`, with no
build command and the build output directory set to `/`. If you ever
recreate it from the dashboard, use exactly those values; there is nothing
to build, and any build command entered there will only make the deployment
worse. Note that the repository name is hyphenated
(`leap-second-preservation-society`) and the Pages project name is not; the
project name matches the domain.

### Custom domain

The domain is not purchased yet, so this section is a promise, not a record.
Once `leapsecondsociety.org` exists, deploy at least once so the project
does too, then:

1. **Add the zone to Cloudflare** (skip if the domain is bought through
   Cloudflare, in which case the zone is already in the account). Dashboard,
   **Add a site**, `leapsecondsociety.org`, Free plan. Cloudflare returns
   two nameservers of the form `xxx.ns.cloudflare.com`.
2. **Repoint the nameservers at the registrar.** Propagation is usually well
   under an hour; Cloudflare emails when the zone goes active.
3. **Attach the domain to the Pages project.** Dashboard, **Workers &
   Pages**, `leapsecondsociety`, **Custom domains**, **Set up a custom
   domain**, `leapsecondsociety.org`. Because the zone is on Cloudflare, the
   proxied CNAME at `@` pointing to `leapsecondsociety.pages.dev` is created
   for you. **Do not create the record by hand first**; a pre-existing CNAME
   blocks the flow outright.
4. **Repeat for `www`** if both should resolve.
5. **Wait for the certificate.** Usually a few minutes.

Until then the site is reachable at `leapsecondsociety.pages.dev`. Then
update the placeholder domain everywhere the table above says to.

### Related

The site's footer files itself as document BEI-007 of
[Best Effort Industries](https://besteffortindustries.com), whose divisions
table does not yet have a row for the Society. Adding one is a manual edit
to that repository's `index.html`; the procedure is in an HTML comment
directly above the table.

## License

Parody. The Society is not a registered charity, horological authority, or
going concern, and holds no rights over any second, past or future. The
facts about TAI, UTC, the IERS and the 2022 CGPM resolution are, regrettably,
accurate.
