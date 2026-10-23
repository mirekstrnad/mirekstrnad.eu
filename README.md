# mirekstrnad.eu

Osobní web Miroslava Strnada — jednostránková statická vizitka
„Miroslav Strnad · Digital / UX / PM“ s kontaktem a odkazy na sociální sítě.

| | |
|---|---|
| Web | http://mirekstrnad.eu (zatím jen HTTP) |
| Hosting | webhosting Gigaserver (DNS `*.ns.gigaserver.cz`), nasazení přes FTP mimo tento repozitář |
| Stack | statické HTML + 2× CSS, Font Awesome 4.1, Open Sans (Google Fonts); žádný JavaScript, žádný build |
| Poslední změna webu | leden 2015 |

## Co je v repozitáři

Repozitář **neobsahuje zdrojové soubory webu** — jen dokumentaci.

```
AGENTS.md              vstup pro AI modely a agenty (co projekt je, co nedělat)
README.md              tento soubor
docs/wiki/             export z domácí wiki (BookStack); zdroj pravdy je wiki
  01-prehled.md        co web je, stav repozitáře, omezení, historie
  02-review.md         kompletní review webu (kód, design, přístupnost,
                       kompatibilita, rychlost, bezpečnost) + doporučení
```

Změny v repozitáři se na webu **neprojeví**. Pokud se má web spravovat z gitu,
je potřeba nejdřív stáhnout stávající soubory z hostingu a zavést nasazení
(viz doporučení v review).

## Struktura webu (podle živé stránky)

```
/index.html                        ~2 kB, lang="cs"
/inc/main.css                      desktop layout (karta absolutně vlevo dole)
/inc/mobile.css                    @media (max-width: 767px)
/font-awesome/                     Font Awesome 4.1.0 (css + fonts)
/design/bg.jpg                     pozadí, 403 kB
/design/photo.jpg                  portrét v kartě (CSS background), 83 kB
/design/pattern.png                překryvný vzor, 1 kB
```

## Stav a doporučení (zkráceně)

Podrobně v [docs/wiki/02-review.md](docs/wiki/02-review.md).

1. **Zapnout HTTPS** — web je jen na `http://`; moderní prohlížeče (Chrome
   s režimem „Always Use Secure Connections“) stránku vůbec nezobrazí nebo
   varují. Zároveň opravit `http://fonts.googleapis.com` (po přechodu na
   HTTPS by jinak šlo o mixed content a písmo by se nenačetlo).
2. **Dostat zdroj do gitu** — jediná kopie je na FTP hostingu.
3. **Opravit layout karty** — fixní `height: 339px; overflow: hidden`
   ořezává obsah při zoomu, fallback fontu nebo delším textu; na mobilu
   `padding-bottom: 700px` tlačí fotku pod ohyb.
4. **Přístupnost** — `maximum-scale=1.0` blokuje zoom, portrét jen jako CSS
   pozadí (bez `alt`), ikony bez `aria-hidden`, dotykové cíle ~20 px.
5. **Rychlost** — ~600 kB přenosu, z toho 403 kB `bg.jpg`; Font Awesome
   (105 kB) kvůli 6 ikonám → inline SVG; obrázky do WebP/AVIF.
6. **Obsah** — odkazy na profily pravděpodobně neplatné
   (`linkedin.com/mirekstrnad/` bez `/in/`, `flickr.com/mirekstrnad/` bez
   `/photos/`, Twitter → X); ověřit a opravit.

## Jak s repozitářem pracovat

- Dokumentace se píše v BookStack wiki a exportuje do `docs/wiki/`
  (formát: YAML frontmatter `title / source / exported / language`, pak Markdown).
- AI agenti: nejdřív [AGENTS.md](AGENTS.md).
- Nezakládat build ani nasazení (GitHub Pages, Cloudflare Pages), dokud o to
  majitel výslovně nepožádá — kolidovalo by to s doménou na hostingu.
- Do repa nepatří přístupy k hostingu ani jiná tajemství.
