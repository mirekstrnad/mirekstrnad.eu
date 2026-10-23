# AGENTS.md — mirekstrnad.eu

Vstup pro AI modely a agenty. Podrobnosti: [docs/wiki/README.md](docs/wiki/README.md),
přehled pro lidi: [README.md](README.md), review a doporučení:
[docs/wiki/02-review.md](docs/wiki/02-review.md).

## Co projekt je

Osobní web Miroslava Strnada (http://mirekstrnad.eu) — jednostránková statická
vizitka „Digital / UX / PM“ s kontaktem a odkazy na sociální sítě.

## Jak to funguje

- Zdroj webu je ve složce `site/` — jeden `index.html` s inline CSS a SVG,
  obrázky v `design/`, fonty ve `fonts/`. Žádný build.
- Push do `master` → GitHub Actions nahraje `site/` na FTP hosting Gigaserver
  (`.github/workflows/deploy.yml`). Přístupy jsou v GitHub Secrets.
- Na serveru se při deployi **nic nemaže** — jsou tam i soubory mimo repo
  (`novaves/`, staré `inc/`, `font-awesome/`).
- Dokumentace: `docs/wiki/` (export z domácí wiki; zdroj pravdy je wiki).

## Co nedělat

- Nedávat do repa přístupy k hostingu ani jiná tajemství.
- Neměnit DNS ani hosting bez ohledu na poštu — na doméně je e-mail
  (MX/SPF u Gigaserveru).
- Nepřidávat JavaScript, externí fonty ani třetí strany — CSP v `.htaccess`
  je `default-src 'none'` + `self`; web je záměrně bez závislostí.
- Nemazat `site/google35b1cd082a048599.html` (Google Search Console).
- Neměnit obsah (text, kontakty, odkazy) bez zadání majitele.

## Stav (23. 10. 2026)

Web běží na https://mirekstrnad.eu (Let's Encrypt, HTTP/2, HSTS, CSP),
nasazuje se automaticky z `master`. Provoz: `docs/wiki/03-provoz.md`;
review původního webu: `docs/wiki/02-review.md`.
