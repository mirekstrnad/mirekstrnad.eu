---
title: mirekstrnad.eu — přepis webu, nasazení a provoz
source: repozitář mirekstrnad/mirekstrnad.eu, 23. 10. 2026
exported: 2026-10-23
language: cs
---

# Přepis webu, nasazení a provoz

Navazuje na [02-review.md](02-review.md). Zdroj webu je od 23. 10. 2026
v repozitáři ve složce `site/` a nasazuje se automaticky.

## Co se změnilo při přepisu (říjen 2026)

| Oblast | Původně (2015) | Nově |
|---|---|---|
| HTML | `div#vcard`, `h2` jako tagline, `h3` sekce, fotka v CSS | `<main><article class="h-card" itemscope Person>`, `h1` + `p.tagline`, sekce `h2`, `<address>`, `<nav>`, fotka jako `<picture>` s `alt` |
| CSS | 2 soubory, fixní `height:339px; overflow:hidden`, `padding-bottom:700px` na mobilu | inline v `<head>` (~4 kB gzip), flex karta řízená obsahem, breakpoint `min-width:900px and min-height:500px`, `clamp()` na `h1`, dotykové cíle 44 px, `:focus-visible`, dark mode, print |
| Viewport | `maximum-scale=1.0` | bez omezení zoomu |
| Ikony | Font Awesome 4.1.0 (105 kB) | 6 inline SVG z Font Awesome Free 7.3.1 (CC BY 4.0), ikona X místo Twitter ptáčka |
| Písmo | Google Fonts přes `http://` | self-host Open Sans 300/400 woff2 (latin + latin-ext), `font-display: swap`, preload |
| Pozadí | `bg.jpg` 403 kB | `bg-{1280,1920,2560}.{avif,webp,jpg}` přes `image-set()`, 1920 AVIF = 23 kB; `pattern.png` nahrazen CSS gradientem |
| Portrét | `photo.jpg` 83 kB jako background | `photo.{avif,webp,jpg}` 8/6/12 kB, `<picture>` |
| Meta | description „Online marketing kvalitně.“, e-mail v `author` | nová description, canonical, OG (og.jpg 1200×630), theme-color, favicon SVG/ICO, apple-touch-icon |
| Odkazy | `http://`, Twitter, `linkedin.com/mirekstrnad/` | `https://`, X, `linkedin.com/in/mirekstrnad/`, `flickr.com/photos/mirekstrnad/` (ověřen), pořadí LinkedIn první |
| Text | bez čárek | „Nevíte, jak…“, „Chcete prostě, aby váš…“ |
| Server | nic | `.htaccess`: nosniff, Referrer-Policy, Permissions-Policy, X-Frame-Options, CSP, cache, gzip, MIME; `robots.txt`, `sitemap.xml` |

Přenos na desktopu: **~600 kB → ~110 kB** (AVIF), 9 → 8 requestů (z toho 4 fonty
s `unicode-range`, všechny ze stejné domény). Mobil ~85 kB.

Ověřeno: `html-validate` bez chyb; vyrenderováno v Chromiu 1440×900 (light/dark),
768×1024, 375×812, 844×390 — karta se nikde neořezává, nadpis na desktopu
na jeden řádek.

**Neověřeno (udělat ručně):** odkazy na LinkedIn, X a Facebook (Flickr ověřen),
HTTP hlavičky po nasazení (securityheaders.com), PageSpeed Insights.

## Nasazení

`.github/workflows/deploy.yml` — při push do `master` se složka `site/` nahraje
na FTP (SamKirkland/FTP-Deploy-Action v4.3.5). Potřebné Secrets a Variables
viz [README](../../README.md#nasazení). Na serveru se **nic nemaže** — staré
soubory (`inc/`, `font-awesome/`, `novaves/`) tam zůstanou; po ověření nového
webu je možné je smazat ručně ve FileZille.

Co na serveru je navíc a **v repu není** (záměrně):

- `novaves/mirek/` — Flash panorama „Pracovna2NP“ (swf, 2010). Flash v
  prohlížečích od 2021 nefunguje; obsah je mrtvý. Zvážit smazání.
- originály `design/bg.jpg` (5184×3456, 403 kB) a `design/photo.jpg` (299×339,
  83 kB) — v repu jsou jen optimalizované varianty; originály si nech u sebe.

## Hosting — zjištěno v administraci (23. 10. 2026)

- Klientská sekce: https://admin.gigaserver.cz (fakturace, služby), technická
  správa: **Administrace** → https://admin-legacy.gigaserver.cz/uzivatel/
- Služba #4406 `mirekstrnad.eu`, tarif Praktik, 150 Kč/rok, zaplaceno do
  4. 12. 2026, auto-prodloužení **Ne** (hlídat!). Kvóta FTP+DB 100 MB
  (využito 26 MB). Na stejném účtu i `strnadmirek.eu` a `danielastrnadova.cz`.
- FTP: `ftp.mirekstrnad.eu`, port 21 (záložní `185.66.36.178`), složka webu
  je v administraci uvedená jako `/www/mirekstrnad.eu`, ale z pohledu FTP účtu
  `mirekstrnad_eu_mirek` je to `/mirekstrnad.eu/` (kořen účtu je o úroveň výš); účty `mirekstrnad_eu`,
  `mirekstrnad_eu_mirek` (2 z 5). Web-FTP: webftp.gigaserver.cz (server WH50).
  FTPS hosting nedokumentuje — deploy workflow má automatický fallback na FTP.
  Zálohy FTP se dělají v neděli (obnova v administraci → FTP).
- **SSL zdarma** (Let's Encrypt) je v Administrace → Ostatní → „SSL zdarma“.
  23. 10. 2026 zapnuto („Vygenerovat a nainstalovat“, bez subdomén — hlavní
  doména + www jsou vždy); stav „Čeká na vygenerování“, hotovo do 24 h.
  Seznam subdomén lze měnit jednou týdně (další změna po 30. 10. 2026).
- DKIM pro SMTP i PHP mail() aktivní. AwStats: http://stats.mirekstrnad.eu.

## HTTPS — stav

✅ Hotovo 23. 10. 2026: certifikát vydán během minut po zapnutí, HTTP/2 funguje.
`.htaccess` přesměrovává `http://` i `www.` na `https://mirekstrnad.eu` (301)
a posílá HSTS `max-age=86400`. securityheaders.com: **A**.

**Úkol cca v lednu 2027:** ověřit, že se certifikát automaticky obnovil
(Let's Encrypt platí 90 dní), a pak zvýšit HSTS na `max-age=31536000`.

## Stav nasazení (23. 10. 2026, večer)

- Nový web je živý: https://mirekstrnad.eu — 3 deploye přes GitHub Actions,
  všechny přes **FTPS** (hosting TLS umí, fallback nebyl potřeba).
- Kořen FTP účtu `mirekstrnad_eu_mirek` obsahuje složku `mirekstrnad.eu/`
  (= web root). První deploy šel omylem do kořene FTP — **28 souborů tam
  zbylo a je třeba je ručně smazat ve FileZille**: `.htaccess`,
  `.ftp-deploy-sync-state.json`, `apple-touch-icon.png`, `favicon.ico`,
  `favicon.svg`, `google35b1cd082a048599.html`, `index.html`, `robots.txt`,
  `sitemap.xml`, složky `design/` a `fonts/` (vše v kořeni, NE v
  `mirekstrnad.eu/`).
- V `mirekstrnad.eu/` zůstaly staré nepoužívané soubory — lze smazat:
  `inc/`, `font-awesome/`, `design/bg.jpg`, `design/photo.jpg`,
  `design/pattern.png`, `novaves/` (Flash). Originály obrázků si nejdřív
  zálohovat (jsou i v `~/Downloads/mirekstrnad.eu`).
- Odkazy na profily ověřeny: X ✅, Facebook ✅, Flickr ✅, LinkedIn ✅
  (`/in/mirekstrnad/` potvrdil majitel).
- Na doméně **nejsou žádné e-mailové schránky** (Administrace → Emaily je
  prázdné); MX/SPF/DKIM existují, ale nic je nepoužívá. Případný přesun
  hostingu tedy poštu neohrozí.

## DNS a pošta

`MX 10 mail.gigaserver.cz`, SPF `v=spf1 mx a include:smtp-gw.gigaserver.cz ~all`.
DMARC chybí (`_dmarc` padá do wildcard CNAME). Protože doména žádnou poštu
neposílá, přidat v Administrace → DNS záznam:
`_dmarc` · TXT · `v=DMARC1; p=reject` (zabrání falšování odesílatele
`@mirekstrnad.eu`). Wildcard `*.mirekstrnad.eu CNAME` zůstává.

## Generování obrázků

```python
from PIL import Image, ImageOps
bg = Image.open('bg.jpg').convert('RGB')
for w in (1280, 1920, 2560):
    im = bg.resize((w, round(bg.height * w / bg.width)), Image.LANCZOS)
    im.save(f'bg-{w}.jpg', quality=72, optimize=True, progressive=True)
    im.save(f'bg-{w}.webp', quality=70, method=6)
    im.save(f'bg-{w}.avif', quality=50, speed=4)
ph = Image.open('photo.jpg').convert('RGB')
ph.save('photo.jpg', quality=82, optimize=True, progressive=True)
ph.save('photo.webp', quality=82, method=6)
ph.save('photo.avif', quality=62, speed=4)
ImageOps.fit(bg, (1200, 630), Image.LANCZOS, centering=(0.6, 0.45)).save('og.jpg', quality=78)
```

Pillow ≥ 11 s AVIF podporou (`pip install pillow`).

## Lokální náhled a kontrola

```
cd site && python3 -m http.server 8000
npx html-validate site/index.html
```

Built-in browser v aplikaci Claude neumí `localhost` — pro vizuální kontrolu
použít Chromium/Playwright nebo nasadit.
