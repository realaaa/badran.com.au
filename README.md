# badran.com.au

Minimal front page for Badran Tech. Hand-written HTML, no build step, no JavaScript,
no external requests. Served by Caddy on web.badran.com.au over both HTTP and HTTPS.

```
public/            everything Caddy serves
  index.html       the page
  index.txt        plain-text version (served to curl/wget at /)
  logo.png         1200px web copy of Badran-Full-Logo-Horizontal-1-052.png
  badran.vcf       contact card
  robots.txt
deploy/
  Caddyfile.snippet
  badran-site-pull.service   git pull --ff-only
  badran-site-pull.timer     every 5 minutes
resources/         original logo files (not served)
```

## Editing

Edit `public/index.html` and `public/index.txt` together — they carry the same facts.
Commit and push to `main`; the server picks it up within ~5 minutes.

## Server setup (one-off, Debian 13)

```sh
# 1. Unprivileged user that owns the checkout
sudo useradd --system --home /srv/badran.com.au --shell /usr/sbin/nologin badran-site

# 2. Clone (public repo: HTTPS clone, no key needed)
sudo git clone https://github.com/realaaa/badran.com.au.git /srv/badran.com.au
sudo chown -R badran-site:badran-site /srv/badran.com.au

#    Private repo instead: add a read-only deploy key for badran-site and clone via
#    git@github.com:realaaa/badran.com.au.git

# 3. Pull timer
sudo cp /srv/badran.com.au/deploy/badran-site-pull.{service,timer} /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now badran-site-pull.timer

# 4. Caddy
#    Add deploy/Caddyfile.snippet to /etc/caddy/Caddyfile, then:
sudo caddy validate --config /etc/caddy/Caddyfile
sudo systemctl reload caddy
```

DNS: `A` (and `AAAA` if the VPS has IPv6) records for both `badran.com.au` and
`www.badran.com.au` set to the VPS static IP (the same address as web.badran.com.au).
The apex can't be a CNAME; `www` could be a CNAME to `badran.com.au` instead.

## Checks

```sh
curl -sI  http://badran.com.au/        # 200, no redirect to https
curl -s   https://badran.com.au/       # plain text
curl -sI  https://badran.com.au/.git/HEAD   # 404
curl -sI  http://www.badran.com.au/    # 301 -> http://badran.com.au/
systemctl list-timers badran-site-pull.timer
journalctl -u badran-site-pull.service -n 20
```
