# badran.com.au

Minimal front page for Badran Tech. Hand-written HTML, no build step, no JavaScript,
no external requests. Served by Caddy on web.badran.com.au over both HTTP and HTTPS.

```
public/            everything Caddy serves
  index.html       the page
  index.txt        plain-text version (served to curl/wget at /)
  logo.png         1200px web copy of Badran-Full-Logo-Horizontal-1-052.png
  badran.vcf       contact card
  robots.txt       blocks AI training crawlers, allows search and AI answers
  sitemap.xml
deploy/
  Caddyfile.snippet
  badran-site-pull.service   git pull --ff-only
  badran-site-pull.timer     daily
resources/         original logo files (not served)
```

## Editing

Edit `public/index.html` and `public/index.txt` together — they carry the same facts.
Commit and push to `main`; the server picks it up within a day. To publish
immediately, on the server:

```sh
sudo systemctl start badran-site-pull.service
```

## Server setup (one-off, Debian 13)

```sh
# 1. Unprivileged user that owns the checkout
sudo useradd --system --home /opt/badran.com.au --shell /usr/sbin/nologin badran-site

# 2. Clone (public repo: HTTPS clone, no key needed)
sudo git clone https://github.com/realaaa/badran.com.au.git /opt/badran.com.au
sudo chown -R badran-site:badran-site /opt/badran.com.au

#    Private repo instead: add a read-only deploy key for badran-site and clone via
#    git@github.com:realaaa/badran.com.au.git

# 3. Pull timer
sudo cp /opt/badran.com.au/deploy/badran-site-pull.{service,timer} /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now badran-site-pull.timer

# 4. Caddy
#    Caddy has no "add site" command: its config is /etc/caddy/Caddyfile.
#    Point it at the snippet in the checkout (once), check, then reload.
echo 'import /opt/badran.com.au/deploy/Caddyfile.snippet' | sudo tee -a /etc/caddy/Caddyfile
sudo caddy validate --config /etc/caddy/Caddyfile   # should mention badran.com.au
sudo systemctl reload caddy
sudo journalctl -u caddy -f   # wait for "certificate obtained successfully", then Ctrl-C
```

Caddy only reads its config on reload: if a pull changes `deploy/Caddyfile.snippet`,
run `sudo systemctl reload caddy` for it to take effect.

DNS: `A` (and `AAAA` if the VPS has IPv6) records for both `badran.com.au` and
`www.badran.com.au` set to the VPS static IP (the same address as web.badran.com.au).
The apex can't be a CNAME; `www` could be a CNAME to `badran.com.au` instead.

## Checks

```sh
curl -sI  http://badran.com.au/        # 200, no redirect to https
curl -s   https://badran.com.au/       # plain text
curl -sI  https://badran.com.au/.git/HEAD   # 404
curl -sI  http://www.badran.com.au/    # 301 -> http://badran.com.au/
curl -s https://badran.com.au/robots.txt
curl -s https://badran.com.au/sitemap.xml
curl -sI -A "Bytespider" https://badran.com.au/            # 403
curl -sI -A "Bytespider" https://badran.com.au/robots.txt  # 200
curl -sI -A "Claude-SearchBot" https://badran.com.au/      # 200
systemctl list-timers badran-site-pull.timer
journalctl -u badran-site-pull.service -n 20
```

## Search engines & listings

Manual, one-off. Verification is by DNS TXT record, so the site has no verification
meta tags or files. Everywhere, use exactly the same name, phone and URL as the page:
Badran Tech, 0410 868 900, https://badran.com.au/.

- **Google Search Console:** add a *domain* property for `badran.com.au`, verify with
  the DNS TXT record, then submit `https://badran.com.au/sitemap.xml`.
- **Bing Webmaster Tools:** import the site from Search Console, then submit the sitemap.
- **Google Business Profile:** service-area business (address hidden), primary category
  "Computer support and services", service areas: the same suburbs as the page
  (Cronulla, Caringbah, Miranda, Sutherland, Kirrawee, Gymea, Jannali, Engadine, Menai,
  Woolooware, Kurnell).
- **Bing Places** and **Apple Business Connect:** set up the same way.
- **Directories:** True Local, Yellow Pages, Hotfrog, Sutherland Shire Chamber of
  Commerce.
