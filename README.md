# KatastarX - novosti nekretnine

## Pristup

Stranica je zaštićena lozinkom (trenutno: davor). Lozinka se pamti u browseru pa je klijent unosi samo prvi put. Za promjenu lozinke: u `index.html` pronađi `atob("ZGF2b3I=")` i zamijeni string rezultatom od base64 kodiranja nove lozinke (npr. na base64encode.org).

Važno: ovo je klijentska zaštita, dovoljna da slučajni posjetitelj ne vidi sadržaj. Repo je public pa tehnički potkovan netko može doći do podataka. Za pravu zaštitu opcije su private repo + Cloudflare Access ili hosting s Basic Auth. Za internu upotrebu i pregled investitorima ovo je sasvim dovoljno.

## Logo

Header trenutno koristi tekstualni logo u stilu brenda (bijeli Katastar + plavi kosi X). Za točni PNG logo: uploadaj `logo.png` (verzija s bijelim tekstom za tamnu pozadinu) u root repozitorija i u `index.html` zamijeni sadržaj `<div class="logo">...</div>` u headeru sa `<img src="logo.png" alt="KatastarX" style="height:34px">`.

Automatski dnevni pregled vijesti s hrvatskog tržišta nekretnina. n8n svaka 24 sata skuplja vijesti, AI ih sažme i generira Facebook pitanje, a rezultat se objavljuje na GitHub Pages linku koji klijent ili investitor otvori jednim klikom.

## Kako radi

1. n8n se pokreće svaki dan u 06:00 (Europe/Zagreb).
2. Čita RSS feedove definiranih izvora.
3. Filtrira samo nekretninske teme iz zadnja 24 sata (ključne riječi: nekretnine, porezi, krediti, najam, APN, HNB i slično).
4. AI sažme svaku vijest na hrvatskom, odredi kategoriju i generira engagement pitanje za Facebook.
5. Rezultat se commita u `news.json` u ovom repozitoriju.
6. GitHub Pages stranica (`index.html`) automatski prikazuje nove kartice.

## Postavljanje, korak po korak

### 1. GitHub repo i Pages (10 minuta)

1. Napravi novi repo, npr. `katastarx-novosti` (public).
2. Uploadaj `index.html` i `news.json` u root repozitorija.
3. Settings, Pages, Source: Deploy from a branch, Branch: `main`, folder `/ (root)`, Save.
4. Za 1-2 minute stranica je živa na `https://TVOJ-USERNAME.github.io/katastarx-novosti/`. To je link za klijenta.

### 2. GitHub token za n8n (5 minuta)

1. GitHub, Settings, Developer settings, Personal access tokens, Fine-grained tokens, Generate new token.
2. Repository access: samo `katastarx-novosti`.
3. Permissions: Contents, Read and write. Ništa drugo.
4. Kopiraj token, trebat će ti u sljedećem koraku.

### 3. n8n Cloud (15 minuta)

1. U n8n: Workflows, Import from File, odaberi `n8n-workflow.json`.
2. Credentials:
   - GitHub node (oba): Create new credential, GitHub API, zalijepi token.
   - OpenAI node: zalijepi svoj OpenAI API ključ (model gpt-4o-mini, trošak oko 1-2 EUR mjesečno za 8 vijesti dnevno).
3. U oba GitHub nodea zamijeni `TVOJ-GITHUB-USERNAME` svojim usernameom.
4. Klikni Execute Workflow za test. Provjeri da se `news.json` u repou promijenio.
5. Aktiviraj workflow (toggle Active).

### 4. Izvori

Feedovi se uređuju u nodeu "Izvori (RSS feedovi)", jedan red po feedu. Trenutno:

| Izvor | Pouzdanost | Napomena |
|---|---|---|
| Index.hr | medijsko | AI provjeri i sažme, izvor označen na kartici |
| Poslovni dnevnik | stručno | |
| tportal | medijsko | |

Službeni izvori (HNB, MPGI, Porezna uprava, DZS, Narodne novine) nemaju uvijek pouzdan RSS. Njihove objave stižu kroz medijske feedove, a mapiranje domena u nodeu "Filter" automatski im dodjeljuje oznaku pouzdanosti ako se link pojavi. Za direktno praćenje službenih stranica dodaj HTTP Request + HTML Extract node, mogu ti to napraviti kao nadogradnju.

Prije aktivacije provjeri da RSS URL-ovi rade (otvori ih u browseru). Ako neki feed promijeni adresu, samo ga zamijeni u nodeu "Izvori".

## Struktura news.json

```json
{
  "updated": "ISO timestamp zadnjeg runa",
  "items": [
    {
      "id": "jedinstveni id",
      "date": "YYYY-MM-DD",
      "category": "cijene | porezi | krediti | zakoni | trziste",
      "reliability": "sluzbeno | strucno | medijsko",
      "title": "naslov",
      "summary": "AI sažetak, 2-3 rečenice",
      "fbQuestion": "gotovo pitanje za Facebook objavu",
      "source": { "name": "ime izvora", "url": "originalni link" }
    }
  ]
}
```

Stranica prikazuje najnovijih 40 vijesti, novije na vrhu. Svaka kartica ima gumb Kopiraj pitanje: jedan klik i FB pitanje je u clipboardu, spremno za Meta Business Suite.

## Troškovi

| Stavka | Cijena |
|---|---|
| GitHub Pages | 0 EUR |
| n8n Cloud | postojeći plan |
| OpenAI (gpt-4o-mini) | oko 1-2 EUR mjesečno |

## Ideje za nadogradnju

1. Tjedni email sažetak investitorima (n8n Send Email node na isti pipeline).
2. Slack ili Telegram notifikacija kad stigne vijest kategorije porezi ili zakoni.
3. Direktni scrape HNB i MPGI objava preko HTTP Request nodea.
