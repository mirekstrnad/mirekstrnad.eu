---
title: mirekstrnad.eu — review webu a doporučení
source: review 23. 10. 2026 (Claude, 2 nezávislí review agenti + ruční ověření)
exported: 2026-10-23
language: cs
---

# Review webu mirekstrnad.eu

Kompletní review živé stránky http://mirekstrnad.eu — kvalita kódu, design
a přístupnost, kompatibilita, rychlost, bezpečnost, hosting. Na konci
prioritizovaná doporučení a odhad pracnosti.

## Jak review vznikalo

- Zdroj webu není v gitu, takže HTML a CSS byly načteny z živé stránky přes
  prohlížeč a ze snapshotu web.archive.org (12/2021, obsah shodný s rokem 2015)
  a zrekonstruovány do souborů pro validaci.
- Dva nezávislé review agenty: (1) kód + přístupnost + kompatibilita,
  (2) bezpečnost + rychlost + hosting. Výsledky sloučeny a ověřeny ručně.
- Validátor `html-validate`, výpočet kontrastů podle WCAG, DNS přes
  dns.google, fakta o Font Awesome, Chrome a Gigaserveru přes web.
- Co **nešlo** ověřit: HTTP hlavičky serveru (síť z pracovního prostředí je
  blokovaná), funkčnost odkazů na sociální profily, Lighthouse měření
  (skóre níže jsou kvalifikované odhady).

## Shrnutí

| Oblast | Stav | Hlavní problém |
|---|---|---|
| Bezpečnost | 🔴 | Jen HTTP, HTTPS nefunguje; Chrome 154+ stránku bez varování nezobrazí |
| Provoz | 🔴 | Jediná kopie zdroje je na FTP hostingu |
| Kompatibilita | 🟠 | Fixní výška karty ořezává obsah; mobilní layout s 700px paddingem; landscape telefony nedostupné |
| Přístupnost | 🟠 | Zoom blokován (`maximum-scale=1`), portrét bez `alt`, 20px dotykové cíle |
| Rychlost | 🟠 | ~600 kB, z toho 403 kB pozadí; 105 kB Font Awesome pro 6 ikon |
| Kvalita kódu | 🟡 | Validní HTML5, ale layout postavený na magic numbers a mrtvém CSS |
| Design | 🟡 | Funguje, ale 2015: tagline jako `h2`, Twitter místo X, bez dark mode/favicon/OG |
| Obsah | 🟡 | Odkazy na profily nejspíš neplatné, chybějící čárky v textu |

## 1. Inventář webu

```
/index.html                 ~2 kB, HTML5, lang="cs", bez JS
/inc/main.css               2,4 kB (gzip 0,7 kB) – desktop
/inc/mobile.css             0,6 kB (bez gzip) – @media (max-width: 767px)
/font-awesome/css/…         Font Awesome 4.1.0, 20,8 kB css + 83,8 kB woff
/design/bg.jpg              403 kB – pozadí (LCP na desktopu)
/design/photo.jpg           83 kB – portrét v kartě (CSS background)
/design/pattern.png         1 kB – překryv s opacity 0.2
Google Fonts                Open Sans 300/400 přes http://fonts.googleapis.com
```

Celkem ~600 kB na desktopu, ~230 kB na mobilu (pozadí se skrývá).
Žádný favicon, canonical, OG/Twitter meta, robots.txt.

DNS: `A 185.66.36.178`, `NS 1–4.ns.gigaserver.cz`,
`MX 10 mail.gigaserver.cz`, `TXT v=spf1 mx a include:smtp-gw.gigaserver.cz ~all`.
`_dmarc` je jen wildcard CNAME na apex → **DMARC fakticky chybí**.
Na doméně **je pošta u Gigaserveru** — každá změna DNS/hostingu to musí zachovat.

## 2. Bezpečnost

| Záv. | Nález | Dopad | Oprava |
|---|---|---|---|
| **Vysoká** | Jen HTTP, HTTPS nefunguje (TLS selhává), žádný HSTS | Chrome s výchozím „Always Use Secure Connections“ (od Chrome 154, 10/2026) zobrazí varování/chybu; Safari/Firefox „Nezabezpečeno“; horší ranking; obsah lze cestou pozměnit | Let's Encrypt u Gigaserveru (nebo Cloudflare), 301 http→https, HSTS (nejdřív `max-age=300`, pak `31536000`) |
| **Střední** | `http://fonts.googleapis.com` | Po přechodu na HTTPS = aktivní mixed content → stylesheet se zablokuje, Open Sans zmizí | Self-host fontu nebo systémový font stack; minimálně `https://` |
| **Střední** | Google Fonts z Googlu | GDPR: LG München 3 O 17493/20 (2022) — dynamické načítání = předání IP do USA bez souhlasu; v ČR malé riziko, ale pro vizitku UX/marketéra zbytečné; Google Fonts od 2020 nesdílí cache mezi weby, výkonový přínos nula | Self-host `woff2` (latin-ext, ~20 kB/řez, `font-display: swap`) nebo `system-ui` stack (0 kB) |
| **Střední** | Žádné security headers (nelze ověřit, u hostingu z 2015 téměř jistě chybí) | Clickjacking (iframe), sniffing, únik refereru | `.htaccess` níže; po nasazení ověřit na securityheaders.com |
| **Nízká** | E-mail v `<meta name="author">` + `mailto:`/`tel:` v textu | Harvestery; kontakt je ale účel vizitky | Smazat e-mail z `meta author`; `mailto:` nechat (UX > obfuskace) |
| **Nízká** | Font Awesome 4.1.0 (2014, EOL) | Žádné CVE pro samotnou knihovnu (CSS+font); nalezená CVE patří WordPress pluginům stejného jména. Riziko = nespravovaná závislost | Odstranit, 6 ikon jako inline SVG |
| **Nízká** | DMARC chybí | Spoofing domény v e-mailu | `_dmarc TXT "v=DMARC1; p=none; rua=mailto:…"`, později `p=quarantine` |

### Navrhovaný `.htaccess` (Apache, sdílený hosting)

```apache
# HTTPS redirect – zapnout až PO vydání certifikátu
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]

<IfModule mod_headers.c>
  Header always set Strict-Transport-Security "max-age=31536000" env=HTTPS
  Header always set X-Content-Type-Options "nosniff"
  Header always set Referrer-Policy "strict-origin-when-cross-origin"
  Header always set Permissions-Policy "camera=(), microphone=(), geolocation=(), payment=()"
  Header always set X-Frame-Options "DENY"
  # statický web bez JS a bez třetích stran (po self-hostu fontů a inline SVG):
  Header always set Content-Security-Policy "default-src 'none'; img-src 'self'; style-src 'self'; font-src 'self'; base-uri 'none'; form-action 'none'; frame-ancestors 'none'; upgrade-insecure-requests"
  # pokud zůstanou Google Fonts: style-src 'self' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com
  Header unset X-Powered-By
  <FilesMatch "\.(css|webp|avif|jpe?g|png|svg|woff2)$">
    Header set Cache-Control "public, max-age=31536000, immutable"
  </FilesMatch>
  <FilesMatch "\.html?$">
    Header set Cache-Control "public, max-age=3600, must-revalidate"
  </FilesMatch>
</IfModule>

<IfModule mod_deflate.c>
  AddOutputFilterByType DEFLATE text/html text/css text/plain image/svg+xml
</IfModule>

AddType font/woff2 .woff2
AddType image/webp .webp
AddType image/avif .avif
Options -Indexes
```

Bloky jsou v `<IfModule>`, takže na hostingu bez `mod_headers` nic nerozbijí.
Na Cloudflare Pages totéž přes soubor `_headers`.

## 3. Rychlost

| Záv. | Opatření | Přínos (odhad) |
|---|---|---|
| **Vysoká** | `bg.jpg` 403 kB → AVIF/WebP + responsive (`image-set()`, varianty 1280/1920/2560), `<link rel="preload" as="image" fetchpriority="high" media="(min-width:768px)">` — obrázek v CSS prohlížeč objeví pozdě | 403 → 90–130 kB (AVIF) / 140–180 kB (WebP); LCP −0,5 až −1,5 s na 4G |
| **Vysoká** | Odstranit Font Awesome, 6 ikon jako inline SVG (~0,3 kB/ikona, `currentColor`) | −105 kB, −2 requesty, bez FOIT ikon, správná ikona X |
| **Střední** | Fonty: systémový stack (0 kB) nebo self-host Open Sans 300+400 woff2 s `font-display: swap` + preload | −1 až −2 round-tripy (300–600 ms na mobilu), −20–40 kB |
| **Střední** | `photo.jpg` → WebP/AVIF přesně 320×339 (+2× retina) | 83 → 25–35 kB |
| **Střední** | Sloučit `mobile.css` do `main.css` a inline do `<head>` (< 3 kB) | −2 render-blocking requesty, FCP −100–300 ms |
| **Střední** | `Cache-Control: immutable` + gzip/brotli (viz `.htaccess`) | Opakovaná návštěva ~0 kB |
| **Nízká** | HTTP/2–3 (vyžaduje HTTPS; na Cloudflare/GitHub Pages automaticky) | Menší efekt při 3–5 requestech |

Po optimalizaci: desktop ~110–230 kB, mobil ~35–70 kB, requesty 9 → 3–5.

| Lighthouse (odhad) | Dnes | Po optimalizaci |
|---|---|---|
| Performance mobil | 70–85 | 95–100 |
| Performance desktop | 80–90 | 97–100 |
| Accessibility | ~85 | 95–100 |
| Best Practices | 70–80 (HTTP) | 100 |
| SEO | 85–90 | 100 |

## 4. Kompatibilita a responzivita

| Záv. | Nález | Oprava |
|---|---|---|
| **Vysoká** | Jen HTTP (viz bezpečnost) — v Chrome 154+ a Safari/Firefox s HTTPS-first se stránka neotevře nebo varuje | HTTPS |
| **Vysoká** | `#vcard { width:390px; height:339px; padding-left:320px; overflow:hidden }` = rozměry `photo.jpg` zakódované do layoutu. Obsah má ~335–345 px, je **na hraně**; fallback font (Open Sans 300 → Arial 400 je širší), text zoom nebo delší e-mail → `h1` na 2 řádky → **kontakty se oříznou** | Fotka jako `<img>`, karta flex/grid bez `height` a `overflow:hidden` |
| **Střední** | `mobile.css`: `height:339px` se neresetuje + `padding-bottom:700px` → box 1044 px bez ohledu na obsah; fotka vždy pod ohybem (~200 px prázdna nad ní); při větším písmu text přeteče přes fotku; fixní `width:290px` → úzký sloupec na 360–767 px, fotka 320 px oříznutá o 10 px po stranách | `#vcard{position:static;height:auto;width:auto;max-width:480px;margin:0 16px;padding:16px}`, `<img width:100%>`, `h1{font-size:clamp(34px,10vw,55px)}` |
| **Střední** | Breakpoint 767 px: iPad portrait (768) dostane desktop — karta 710 px nesymetricky, 20px dotykové cíle. **Landscape telefony** (iPhone 844×390, Pixel 915×412) také desktop: karta potřebuje 369 px výšky, viewport má ~300–340 → `h1` nad viewportem a **nejde k němu odscrollovat** (absolute + `body{height:100%}`) | `@media (min-width:900px) and (min-height:500px)` pro desktop layout; nebo `position:fixed; max-height:calc(100dvh - 60px); overflow:auto` |
| **Střední** | Font Awesome 4.1.0 (5/2014): funguje, ale řada 4.x EOL (4.7.0, 2016), aktuální 7.x; bez woff2 a `font-display` → FOIT až 3 s; ikona `fa-twitter` = starý ptáček, X je až od FA 6.4 | Inline SVG |
| **Nízká** | Vypnuté/selhané webfonty → `h1` se zalomí → ořez (viz výše); FA bez fontu = 6 „tofu“ čtverečků | Zrušit fixní výšku, `font-display: swap`, fallback stack |
| **Nízká** | Dark mode: nic (`body{background:#fff}` natvrdo) | `:root{color-scheme:light dark}` + `@media (prefers-color-scheme:dark)` — pro ČB fotku snadné |
| **Nízká** | Tisk: oba CSS jsou `media="screen"` → nestylované HTML, bez fotky, odkazy bez URL | `main.css` `media="all"` + `@media print` blok |

## 5. Přístupnost (WCAG 2.1 AA)

Kontrasty (spočteno): `h2 #5f5f5f` na bílé = **6,39:1** (AA ✓, AAA ✗; pro
AAA `#595959`); odkazy `#07548c` = **7,91:1** (AA i AAA ✓); odkaz vs. okolní
černý text 2,66:1 bez podtržení — formálně OK (seznam s ikonami), ale podtržení
doporučeno.

| Záv. | Nález | Kritérium | Oprava |
|---|---|---|---|
| **Vysoká** | `maximum-scale=1.0` blokuje pinch-zoom na Androidu | 1.4.4 | `<meta name="viewport" content="width=device-width, initial-scale=1">` |
| **Vysoká** | Fixní výška + `overflow:hidden` → při text-only zoomu (Firefox) a Text Spacing (+~40 % obsahu) kontakty zmizí | 1.4.4, 1.4.12 | Zrušit fixní výšku |
| **Střední** | Portrét jen jako CSS pozadí — je to **obsah** (identifikuje osobu), ne dekorace; `bg.jpg` dekorace zůstat může | 1.1.1 | `<img src="design/photo.jpg" width="320" height="339" alt="Miroslav Strnad, portrétní fotografie">` |
| **Střední** | FA ikony `<span class="fa">` bez `aria-hidden`; zbytečné dvojité vnoření | 1.1.1 / 4.1.2 | `aria-hidden="true"` / inline SVG s `aria-hidden focusable="false"` |
| **Střední** | Dotykové cíle ~20×60 px (inline `li`); na mobilu padding na `li`, ne na `a` | 2.5.5 / 2.5.8 (WCAG 2.2 AA 24 px) | `ul li a{display:inline-flex;align-items:center;min-height:44px}` |
| **Střední** | Žádné landmarky (`main`, `header`, `address`, `nav`) | 1.3.1 / 2.4.1 | `<main>`, `<address>`, `<nav aria-label="Profily">`; prázdné `div#pattern/#background` → pseudo-elementy |
| **Nízká** | `:focus-visible` nestylovaný, `:hover` podtrhává, `:focus` ne | 2.4.7 | `a:hover,a:focus-visible{text-decoration:underline}` + outline |
| **Nízká** | `tel:` s mezerami (validátor `tel-non-breaking`) | — | `white-space: nowrap` nebo `&nbsp;` |

## 6. Kvalita kódu

HTML5 validní (`html-validate`: jen 2× `tel-non-breaking`), CSS syntakticky OK.

| Záv. | Nález | Oprava |
|---|---|---|
| **Vysoká** | Layout na magic numbers (`390/339/320 px`, `z-index: 9999/100`, `bottom/left: 30px`) | Flex karta řízená obsahem, `:root` custom properties |
| **Střední** | Osnova nadpisů: tagline „Digital / UX / PM“ jako `h2`, sekce kontaktů jako `h3` pod ním | `<p class="tagline">` + sekce jako `h2` |
| **Střední** | Žádné strukturované údaje — vizitka je ideální pro h-card / schema.org `Person` | `<article class="h-card" itemscope itemtype="https://schema.org/Person">`, `p-name`, `u-photo`, `u-tel`, `u-email`, `u-url rel="me"` / `itemprop="sameAs"` |
| **Střední** | Chybí favicon, canonical, OG/Twitter meta, `theme-color` | 404 na `/favicon.ico` při každé návštěvě; sdílení bez náhledu; duplicitní URL http/https/www |
| **Nízká** | Mrtvé CSS: `list-style-*` na inline `li` (marker se nevykreslí), `ol, address` bez markupu, `html,body{border:0}`, duplicitní `inset:0` + `width/height:100%`, `mobile.css` opakuje `h1{font-size:55px}`, `type="text/css"`, `meta robots all,follow` (= default) | Smazat |
| **Nízká** | Hacky z 2013: `text-shadow: 1px 1px 1px rgba(0,0,0,.004)`, `-webkit-font-smoothing: antialiased` | Smazat (dnes bez efektu, resp. vynutí paint layer) |
| **Nízká** | Odkazy `http://` na profily s pravděpodobně neplatnými cestami: `linkedin.com/mirekstrnad/` (LinkedIn používá `/in/…`), `flickr.com/mirekstrnad/` (`/photos/…` nebo `/people/…`), `twitter.com` → `x.com` | Ověřit ručně a opravit na finální `https://` URL |
| **Nízká** | Čeština: „Nevíte, jak…“, „Chcete prostě, aby…“ — chybějící čárky | Doplnit |

## 7. Design

Co funguje: silná, jednoduchá kompozice (velký ČB portrét + bílá karta),
jeden jasný účel (kontakt), žádný balast, žádný JS, Open Sans 300 v nadpisu
působí dodnes čistě.

Co zestárlo:

- Tagline „Digital / UX / PM“ a text „Online marketing kvalitně“ v `description`
  nejsou v souladu (description je z dřívější verze webu).
- Twitter ikona a název — síť je dnes X; Flickr a Facebook zvážit, jestli jsou
  ještě relevantní profily; LinkedIn je pro „Digital / UX / PM“ nejdůležitější
  a je až poslední.
- Karta přilepená vlevo dole s pevnými rozměry — na širokých monitorech
  (2560+) je drobná, na tabletech nesymetrická.
- Bez dark mode, bez favicon, bez OG náhledu — sdílený odkaz vypadá prázdně.
- Mobil: fotka až pod ohybem, nad ní prázdno; na mobilu by měla být
  fotka nahoře nebo vedle jména.

Návrh (bez změny charakteru): zachovat ČB portrét + bílou kartu, kartu
udělat obsahově řízenou (flex), na mobilu fotka jako `<img>` nahoře v kartě,
`clamp()` pro `h1`, přidat dark variantu (`#111` pozadí, `#1c1c1c` karta,
odkazy `#7fb6e6` ≈ 8,5:1), favicon jako SVG monogram „MS“, OG obrázek
= `photo.jpg`.

## 8. Hosting a provoz

**Riziko: zdroj webu není v gitu.** Jediná kopie na FTP + archiv. Ztráta
přístupu k hostingu = ztráta webu. Oprava (1 h): stáhnout FTP → commit do
`mirekstrnad/mirekstrnad.eu` → nasazovat z repa.

Gigaserver uvádí Let's Encrypt jako bezplatnou volbu (gigaserver.cz/ssl-certifikaty;
hosting od ~120 Kč/rok bez DPH), ale **„zdarma v ceně hostingu“ výslovně nepíše** —
ověřit v administraci/na podpoře. HTTP/2, brotli, `mod_headers` neuvádějí.

| Varianta | Cena | HTTPS | Pracnost | Rizika |
|---|---|---|---|---|
| **(a) Gigaserver + Let's Encrypt** | stávající | LE z administrace, auto-renew; HTTP/2 nejisté | **0,5–1 h** | Nejmenší — DNS, MX, SPF beze změny; `mod_headers` neověřen; deploy dál FTP (řešit GitHub Action) |
| **(b) Cloudflare Pages s vlastní doménou** | 0 Kč hosting, ale Gigaserver **zůstává kvůli poště** → úspora ~0 | automaticky, HTTP/3, brotli, `_headers` | **2–3 h** | Apex doména = přesun NS na Cloudflare → ručně přenést MX + SPF (+ DMARC); výpadek pošty při chybě; známý workflow (pozemky) |
| **(b') GitHub Pages** | 0 Kč | automaticky | 1–2 h | Apex přes A záznamy bez přesunu NS, ale **bez vlastních headers** (CSP/HSTS nejde) |
| **(c) Cloudflare proxy před Gigaserverem** | 0 Kč | browser↔CF HTTPS; CF↔origin HTTP („Flexible“) dokud není LE na originu | 1–2 h | Stejné DNS riziko jako (b); dvojí údržba; smysl jen jako mezikrok |

**Doporučení:** pokud Gigaserver zapne LE bez příplatku → **(a)** + zdroj do
gitu + FTP deploy akce. Pokud chceš sjednotit workflow s projektem pozemky →
**(b) Cloudflare Pages**, ale před přesunem NS vyexportovat celou zónu
(A, MX, TXT/SPF, CNAME pro webmail/autodiscover) a po přesunu ověřit
doručení pošty. V obou případech přidat DMARC.

## 9. Doporučení — prioritizovaný plán

| # | Co | Proč první | h |
|---|---|---|---|
| 1 | **HTTPS** (LE u Gigaserveru nebo CF), 301, HSTS; zároveň pryč `http://` Google Fonts | Bez toho web v Chrome 154+ „neexistuje“; mixed content by po přechodu rozbil písmo | 0,5–1 |
| 2 | **Zdroj do gitu** + deploy z repa (FTP action / Pages) | Jediná kopie na FTP | 1 |
| 3 | **Přepis HTML**: `<img>` portrét s `alt`, `<main>/<address>/<nav>`, h-card/schema.org, `h1` + tagline `p` + sekce `h2`, viewport bez `maximum-scale`, favicon, canonical, OG, `aria-hidden` na ikonách, opravené social odkazy (X, `/in/`), čárky v textu, smazat e-mail z `meta author` | Řeší přístupnost, SEO, sdílení najednou | 2–3 |
| 4 | **Přepis CSS do jednoho souboru (inline)**: flex karta bez fixní výšky, breakpoint `min-width:900px and min-height:500px`, `clamp()` na `h1`, dotykové cíle 44 px, `:focus-visible`, dark mode, print; smazat mrtvý kód a hacky | Odstraní ořez obsahu, nedostupný `h1` na landscape, 700px padding | 3–4 |
| 5 | **Font Awesome → 6 inline SVG; Open Sans self-host woff2 nebo systémový stack** | −140 kB, bez FOIT, bez třetích stran (GDPR, CSP) | 1–2 |
| 6 | **Obrázky**: `bg.jpg`/`photo.jpg` → AVIF+WebP, `image-set()`, preload LCP | 67 % přenosu | 1–1,5 |
| 7 | **`.htaccess` / `_headers`**: security headers, cache, gzip/brotli; DMARC v DNS | Best Practices 100, CSP triviální pro web bez JS | 1 |
| 8 | **Test**: iOS/Android/iPad/landscape, zoom 200 %, text-spacing bookmarklet, securityheaders.com, PageSpeed, ověření odkazů | Potvrzení | 1–2 |

**Celkem ~10–15 h** (varianta a). Body 1–2 jsou samostatné rychlé výhry
(~2 h) a dají se udělat hned bez zásahu do designu.

## Zdroje

- [Google Security Blog — HTTPS by default (10/2025)](https://security.googleblog.com/2025/10)
- [BleepingComputer — Chrome to warn before opening HTTP sites](https://bleepingcomputer.com/news/google/google-chrome-to-warn-users-before-opening-insecure-http-sites)
- [endoflife.date — Font Awesome](https://endoflife.date/font-awesome)
- [Gigaserver — SSL certifikáty](https://www.gigaserver.cz/ssl-certifikaty/), [webhosting](https://www.gigaserver.cz/webhosting/)
- [LG München 3 O 17493/20 — Google Fonts a GDPR](https://www.ra-plutte.de/lg-muenchen-dynamische-einbindung-google-web-fonts-ist-dsgvo/)
- FA „CVE“ patří WP pluginům: [CVE-2022-3829](https://nvd.nist.gov/vuln/detail/cve-2022-3829), [CVE-2023-0419](https://nvd.nist.gov/vuln/detail/cve-2023-0419), [CVE-2023-4718](https://www.rapid7.com/db/vulnerabilities/cve-2023-4718/)
- Snapshot webu: web.archive.org, 2021-12-18 (shodný s živou verzí z ledna 2015)
