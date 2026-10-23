# AGENTS.md — mirekstrnad.eu

Vstup pro AI modely a agenty. Podrobnosti: [docs/wiki/README.md](docs/wiki/README.md),
přehled pro lidi: [README.md](README.md), review a doporučení:
[docs/wiki/02-review.md](docs/wiki/02-review.md).

## Co projekt je

Osobní web Miroslava Strnada (http://mirekstrnad.eu) — jednostránková statická
vizitka „Digital / UX / PM“ s kontaktem a odkazy na sociální sítě.

## Důležité: repozitář neobsahuje zdroj webu

- Web běží na webhostingu **mimo GitHub** a nasazuje se tam mimo tenhle repozitář.
- V repu je jen dokumentace (`AGENTS.md`, `docs/wiki/`). Není tu žádný build,
  žádné nasazení, žádný kód.
- Změny v repozitáři se na webu **neprojeví**.

## Co nedělat

- Nevymýšlet obsah ani strukturu webu — zdrojové soubory tu nejsou; když mají
  přibýt, dodá je majitel.
- Nezakládat build ani nasazení (GitHub Pages, Cloudflare Pages), dokud o to
  majitel výslovně nepožádá — mohlo by to kolidovat s doménou na hostingu.
- Nedávat do repa přístupy k hostingu ani jiná tajemství.

## Známý stav webu (23. 10. 2026)

Web je jen na HTTP (HTTPS nefunguje), zdroj je pouze na FTP hostingu
Gigaserver, na doméně je pošta (MX/SPF u Gigaserveru) — každý návrh změny
DNS/hostingu to musí zachovat. Nálezy a plán oprav: `docs/wiki/02-review.md`.
