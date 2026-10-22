---
title: mirekstrnad.eu — přehled
source: BookStack Projekty, page 79
exported: 2026-10-22
language: cs
---

# mirekstrnad.eu

Mirkův osobní web — jednostránková vizitka „Miroslav Strnad · Digital / UX / PM“
s krátkým textem, kontaktem a odkazy na sociální sítě.

| | |
|---|---|
| Web | http://mirekstrnad.eu (jen HTTP) |
| Repozitář | https://github.com/mirekstrnad/mirekstrnad.eu — veřejný, založený 2014, popis „Osobní web.“ |

## Repozitář neobsahuje zdroj webu

Až do 22. 10. 2026 byl repozitář úplně prázdný (žádný commit, žádná větev).
Teď obsahuje jen tuhle dokumentaci a `AGENTS.md`. Zdrojové soubory webu na GitHubu
**nejsou**; web běží na webhostingu mimo GitHub. Jestli kód existuje i jinde
(počítač, FTP u hostingu), z repa poznat nejde.

## Co je vidět na webu (ověřeno 22. 10. 2026)

- web odpovídá normálně (`200 OK`),
- stránka se naposledy změnila v lednu 2015,
- statické HTML (~2 kB) se dvěma styly (hlavní a mobilní), Font Awesome a font
  Open Sans z Google Fonts,
- obsah: jméno, „Digital / UX / PM“, nabídka spolupráce, kontakt, odkazy na
  Twitter, Facebook, Flickr a LinkedIn,
- **HTTPS nefunguje**, web je jen na `http://`.

## Omezení a co by šlo udělat

- Web je přes 10 let starý a bez HTTPS; prohlížeče ho označí „nezabezpečeno“.
- Odkaz na LinkedIn má tvar `linkedin.com/mirekstrnad/` (bez `/in/`), nejspíš
  nevede na profil. Twitter je dnes X.
- Kdyby se měl web udržovat v gitu: nahrát stávající soubory do repa a nasadit
  stejně jako ostatní projekty (GitHub Pages / Cloudflare Pages s vlastní doménou
  → zadarmo i HTTPS). Nebo repo smazat, ať neplete.

## Historie

| Datum | Co |
|---|---|
| 2014 | založen repozitář (prázdný) |
| leden 2015 | poslední změna stránky na hostingu |
| 22. 10. 2026 | do repozitáře přidána dokumentace (`AGENTS.md`, `docs/wiki/`) |
