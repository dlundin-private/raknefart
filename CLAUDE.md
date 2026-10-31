# Räknefart

Ett mattespel för barn (6–13 år) där man räknar så snabbt man kan och tävlar mot sitt eget rekord. Syskonspel till Läsfart. Hobbyprojekt, spelas av familjen på mobil, surfplatta och dator.

## Principer (bryt inte mot dessa)
- All text som barnen ser ska vara på svenska. Decimalkomma i tider (34,2).
- Inga annonser, inget pay-to-win, inga köp. Man tävlar mot sig själv, aldrig mot andra spelare.
- Ingen Cloudflare och ingen backend. Allt ligger i git.
- Inget byggsteg. Hela spelet är `index.html` med CSS och JS inline. Filerna i roten publiceras exakt som de är, och `index.html` går att öppna direkt i en webbläsare (då utan offlineläge).
- Inget laddas externt. Typsnitten Fredoka och Atkinson Hyperlegible ligger i `fonts/` (SIL OFL, licenstexten bredvid) och laddas via `@font-face`. Länka inte till Google Fonts, det skickar besökarens IP-adress till Google (GDPR). Spelet ska fungera med reservtypsnitten om typsnitten inte laddar.
- Spelet ska fungera offline.

## Filer
- `index.html` — hela spelet.
- `sw.js` — service worker som sparar alla filer för offlineläge.
- `manifest.webmanifest` och `icons/` — gör att spelet kan läggas på hemskärmen.
- `fonts/` — typsnitten (woff2) med sina OFL-licenser.
- `CNAME` — egen domän för GitHub Pages.
- `CLAUDE.md` — den här filen.

## Offlineläge
`sw.js` sparar filerna i listan `FILES` vid första besöket och svarar sedan från cachen. **Höj `VERSION` i `sw.js` vid varje ändring av en publicerad fil**, annars fortsätter installerade kopior att visa den gamla versionen. Lägg till nya filer i `FILES`.

## Spelregler
- Ett lopp = 20 rätta svar (`GOAL`).
- Svaret rättas automatiskt när rätt antal siffror är inskrivet. Ingen enter-knapp.
- Fel svar: +3 sekunder (`PENALTY`), rätt svar visas i 1,1 s, uppgiften kommer tillbaka 3 uppgifter senare.
- Missade uppgifter sparas per spelare och bana (max 12) och dyker upp i ca 30 % av uppgifterna i senare lopp. De tas bort när de besvaras rätt på första försöket.
- Spöket ("R") springer i takt med spelarens rekord på banan.
- Stjärnor: 1 för att gå i mål, 2 och 3 om snittiden per tal är under banans gränser `t: [tvåStjärnor, treStjärnor]` (sekunder).
- Alla banor är öppna. Ingen upplåsning.

## Banor
Banorna ligger i arrayen `LEVELS`. Varje bana:
```js
{ id:'gg1', name:'Gånger 2, 5 och 10', hint:'Från 8 år', t:[4,2.5], gen(){ return P('3 × 4', 12); } }
```
- `gen()` returnerar `P(fråga, svar, blank)`. Med `blank = true` markerar `?` i frågan var svaret ska stå (t.ex. `'7 + ? = 10'`). Annars visas `fråga = ?`.
- Använd riktiga tecken: `−` (minus), `×`, `÷`.
- Svaren måste vara heltal ≥ 0 och högst 4 siffror.
- Ändra aldrig en befintlig banas `id`, då försvinner sparade rekord.
- `mix` plockar slumpvis från övriga banor utom de lättaste. Uppdatera filtret om nya banor läggs till.

## Sparad data
`localStorage`, nyckel `raknefart-v1`, alltid inom try/catch:
```js
{ players:[{ id, name, levels:{ [levelId]: { best: ms|null, stars: 0-3, missed: [{q,a,blank}] } } }], current: playerId }
```
Barnens rekord får inte försvinna. Om strukturen ändras: lägg till en migrering som läser gammal data, och byt bara nyckel om det är nödvändigt.

## Design
- Färger är CSS-variabler på `:root` med mörkt läge via `prefers-color-scheme` och `data-theme`. Använd alltid variablerna.
- Fredoka för siffror och rubriker, Atkinson Hyperlegible för brödtext (lättläst för barn).
- Löparbanan (blå bana, gul spelarbricka, rutig målflagga) är spelets kännetecken.
- Respektera `prefers-reduced-motion`. Siffertangenterna reagerar på `pointerdown` för snabbhet, och tangentbordet fungerar på dator (siffror, Backspace, Escape).
- Viktigt: element som döljs med attributet `hidden` men har `display` satt i CSS behöver en `[hidden]{display:none}`-regel, annars syns de ändå (det var orsaken till en bugg där "Kör!" låg kvar över uppgiften).

## Testa efter ändringar
1. Syntaxkontroll: `python3 -c "import re;s=open('index.html').read();open('/tmp/a.js','w').write(re.search(r'<script>(.*)</script>',s,re.S).group(1))" && node --check /tmp/a.js`
2. Öppna `index.html` i webbläsaren: skapa en spelare, kör ett lopp, svara fel minst en gång, kontrollera resultatsidan, kör igen så spöket syns.
3. Kolla smal mobilvy (ca 375 px) och både ljust och mörkt läge.
4. Offline (via en lokal server, t.ex. `npx serve .`): ladda sidan, slå av nätet i DevTools och ladda om.

## Publicering
GitHub Pages publicerar grenen `main` från roten (repo `dlundin-private/raknefart`). En push till `main` är live efter någon minut på https://raknefart.chandler.se (filen `CNAME`, DNS-posten pekar på `dlundin-private.github.io`).

Sparningen i `localStorage` hör ihop med domänen. Byt inte domän, då börjar alla spelare om från noll.
