# mirekstrnad.eu

Osobní web Miroslava Strnada — jednostránková statická vizitka
„Miroslav Strnad · Digital / UX / PM“ s kontaktem a odkazy na profily.

| | |
|---|---|
| Web | https://mirekstrnad.eu (HTTPS po zapnutí certifikátu u hostingu) |
| Hosting | webhosting Gigaserver (DNS, pošta i web), nasazení přes FTP z GitHub Actions |
| Stack | jedno HTML s inline CSS, self-hostovaný Open Sans (woff2), inline SVG ikony; žádný JavaScript, žádný build |

## Struktura repozitáře

```
site/                      zdroj webu – přesně to, co se nahrává na hosting
  index.html               stránka (HTML + inline CSS + SVG ikony)
  .htaccess                security headers, cache, gzip, MIME typy
  design/                  bg-{1280,1920,2560}.{avif,webp,jpg}, photo.{avif,webp,jpg}, og.jpg
  fonts/                   Open Sans 300/400, latin + latin-ext (SIL OFL)
  favicon.svg, favicon.ico, apple-touch-icon.png
  robots.txt, sitemap.xml
  google35b1cd082a048599.html   ověření Google Search Console – nemazat
.github/workflows/deploy.yml   push do master → FTP deploy složky site/
docs/wiki/                 dokumentace (export z domácí wiki): přehled, review, provoz
AGENTS.md                  vstup pro AI modely a agenty
```

## Nasazení

Každý push do `master`, který změní `site/`, spustí workflow **Deploy to FTP**
([SamKirkland/FTP-Deploy-Action](https://github.com/SamKirkland/FTP-Deploy-Action)).
Nahrávají se jen změněné soubory; **nic se na serveru nemaže**
(`dangerous-clean-slate: false`), takže staré soubory (`inc/`, `font-awesome/`,
`novaves/`) zůstávají, dokud je někdo neodstraní ručně.

Nastavení v GitHubu (Settings → Secrets and variables → Actions):

| Typ | Název | Hodnota |
|---|---|---|
| Secret | `FTP_PASSWORD` | heslo FTP účtu |
| Variable | `FTP_USER` | nepovinné, výchozí `mirekstrnad_eu_mirek` |
| Variable | `FTP_HOST` | nepovinné, výchozí `ftp.mirekstrnad.eu` |
| Variable | `FTP_SERVER_DIR` | nepovinné, výchozí `/` (kořen FTP účtu = `/www/mirekstrnad.eu`) |

Workflow zkusí FTPS; když hosting TLS neumí, automaticky použije FTP.

Ruční spuštění: Actions → Deploy to FTP → Run workflow.

## Lokální náhled

```
cd site && python3 -m http.server 8000   # http://localhost:8000
```

## Jak dělat změny

1. Upravit `site/index.html` (vše je v jednom souboru).
2. Obrázky: originál `bg.jpg` (5184×3456) a `photo.jpg` (299×339) nejsou v repu
   (jsou na hostingu / u majitele); varianty v `design/` se generují Pillow
   (AVIF q50, WebP q70, JPEG q72) — postup v `docs/wiki/03-provoz.md`.
3. Commit do větve → PR → merge do `master` → automatické nasazení.

## Dokumentace

- [docs/wiki/01-prehled.md](docs/wiki/01-prehled.md) — co web je, historie
- [docs/wiki/02-review.md](docs/wiki/02-review.md) — review původního webu (10/2026) a doporučení
- [docs/wiki/03-provoz.md](docs/wiki/03-provoz.md) — co se změnilo při přepisu, nasazení, HTTPS, údržba
- [AGENTS.md](AGENTS.md) — pokyny pro AI agenty
