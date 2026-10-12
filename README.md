# My Orchids Web & Email Architecture

Detta repository innehåller källkoden till den tvåspråkiga landningssidan för mobilappen **My Orchids** (`myorchids.se`). Webbsidan är byggd som en statisk webbplats med Jekyll och Bootstrap 5, och hela infrastrukturen är optimerad för att köras helt utan löpande månadskostnader.

---

## 🗺️ Systemarkitektur & Flödesschema

Här beskrivs hur de olika systemen samverkar för att hantera både webbtrafik och e-post helt kostnadsfritt efter avvecklingen av det traditionella webbhotellet.

### 1. Webbtrafik (HTTP/HTTPS)
```text
[Besökare] ➡️ (myorchids.se) ➡️ [Simply.com (Registrator)] ➡️ [Cloudflare (DNS Only)] ➡️ [GitHub Pages] ➡️ (Visar din Jekyll-sida)
```

### 2. Inkommande E-post (Mottagning)
```text
[Avsändare] ➡️ (info@myorchids.se) ➡️ [Cloudflare Email Routing (MX)] ➡️ [Gmail (myorchids.se@gmail.com)]
```

### 3. Utgående E-post (Svara/Skicka)
```text
[Gmail Gränssnitt] ➡️ [SMTP2GO (Externt Alias via TLS/587)] ➡️ [Mottagare (Verifierat med DKIM/SPF)]
```

---

## 🏗️ Komponentansvar & Systemroller

### 1. Simply.com (Domänregistrator & Grund-DNS)
* **Ansvar:** Hanterar enbart äganderätten och den årliga förnyelsen av domännamnet `myorchids.se`.
* **Konfiguration:** Ändrad från *Webbhotellspaket* till en helt kostnadsfri **DNS Service**. **DNSSEC** är permanent inaktiverat (`unsigned delegation`) för att tillåta externa servrar.
* **Namnservrar:** Pekar om domänansvaret till Cloudflare via följande två adresser:
  * `harleigh.ns.cloudflare.com, joel.ns.cloudflare.com`

### 2. Cloudflare (DNS-hantering & Mailsluss)
* **Ansvar:** Fungerar som sajtens primära DNS-växel samt hanterar all inkommande e-post.
* **Webb-DNS:** Innehåller 4 stycken `A`-records som slussar huvuddomänen till GitHub Pages, samt ett `CNAME` för `www`. Dessa är inställda på **Proxied** (orange moln).
* **Email Routing:** Inkommande trafik till `info@myorchids.se` och `admin@myorchids.se` fångas upp av Cloudflares `MX`-poster och vidarebefordras automatiskt till det privata Gmail-kontot.

### 3. GitHub Repository (`myorchids-website`)
* **Ansvar:** Versionshanterar källkoden till webbplatsen.
* **Innehåll:** Innehåller Jekylls filstruktur, Bootstrap 5-tillgångar (lokalt sparade `.css` och `.js` i `assets/`), samt en **`CNAME`**-fil i roten med texten `myorchids.se`.

### 4. Jekyll (Statisk Webbplatsmotor)
* **Ansvar:** Genererar och bygger ihop webbplatsens sidor lokalt och på servern.
* **Struktur:** 
  * `index.html`: Engelska startsidan (standard/default).
  * `sv.html`: Svenska sidan (tillgänglig på `myorchids.se/sv/` via permalänk).
  * `_layouts/default.html`: Huvudmallen som laddar in lokala Bootstrap-filer, menyer och sidfötter.
  * `_config.yml`: Systemets kontrollpanel där `url` är satt till `https://myorchids.se` och `baseurl` är lämnad helt tom `""`.

### 5. GitHub Pages (Webbhotell & Hosting)
* **Ansvar:** Driftar hemsidan på internet helt gratis och hanterar det automatiska krypteringscertifikatet (**Enforce HTTPS**).
* **Verifiering:** Domänen är låst och verifierad till ditt specifika GitHub-konto för att förhindra domänkapning.
* Site publiceras automatiskt när man pushar förändring till main-branchen

### 6. Gmail (`myorchids.se@gmail.com`)
* **Ansvar:** Slutstationen för all e-posthantering. Fungerar som den centrala inkorgen där du läser, organiserar och besvarar alla mail.

### 7. SMTP2GO (Utgående E-postmotor)
* **Ansvar:** Fungerar som den utgående SMTP-servern (`mail.smtp2go.com` via Port 587/TLS).
* **Säkerhet:** Verifierad via unika **CNAME**-rader i Cloudflare för att aktivera **DKIM** och **SPF**, vilket garanterar att dina svar till kunder inte klassas som skräppost.

---

## 🛠️ Lokal utveckling & Underhåll

Om du vill göra ändringar på hemsidan i framtiden, följer här instruktionerna för hur du kör projektet lokalt på din Windows-dator (optimerat för Ruby 4.0/Jekyll 4.3+ krockar):

### Konfiguration i `Gemfile`
Din `Gemfile` är uppstädad och moderniserad för att slippa äldre miljöfel:
```ruby
source "https://rubygems.org"
gem "jekyll", "~> 4.3"
gem "webrick"
gem "wdm", ">= 0.1.0" # Gör övervakningen snabb på Windows
```

### Kommandon för terminalen
1. **Rensa gammal cache och byggmapp:**
   ```bash
   bundle exec jekyll clean
   ```
2. **Starta den lokala servern:**
   ```bash
   bundle exec jekyll serve
   ```
3. Öppna webbläsaren på: `http://127.0.0.1:4000/`
4. Gör dina ändringar i koden, spara och tryck **Ctrl + F5** i webbläsaren för att se uppdateringen live.

---

## 🔒 Säkerhetsinställningar i Gmail
För att svara automatiskt från rätt adress i Gmail:
1. Gå till **Inställningar** ➡️ **Konton och import** ➡️ **Skicka e-post som**.
2. Kontrollera att både `info@` och `admin@` har rutan **Behandla som ett alias** ibockad.
3. Se till att radioknappen **"Svara från samma adress som meddelandet skickades till"** är permanent vald längst ner i samma sektion.

---
*Detta arkiv och dess infrastruktur är upprättat i oktober 2026.*
