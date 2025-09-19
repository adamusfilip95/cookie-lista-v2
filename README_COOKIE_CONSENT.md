# Mojra – Cookie Consent + GA4 (Consent Mode V2)

Implementace cookie banneru pro Mojra.cz s podporou Google Consent Mode V2 a GA4.

---

## 🎯 Cíl
- Nasadit **jediný balíček** s Consent Mode V2 + GA4.
- Odstranit starý **UA kód** i případné duplicity **gtag.js**.
- Zajistit správné pořadí skriptů, aby se `consent default` spustil dřív, než se načte Google Analytics.

---

## 📦 Verze souborů

Repozitář obsahuje dvě varianty:

- **`cookie_consent_full.min.html`**  
  → minifikovaná produkční verze.  
  → **tuto nasazujte na web** (rychlá, optimalizovaná).

- **`cookie_consent_full.pretty.html`**  
  → formátovaná verze pro čitelnost.  
  → slouží k **code review** a případným úpravám programátorem.  

- **`README.md`**  
  → tento návod (implementace, postup, testování).

---

## 🔧 Postup nasazení

### 1) Odstranit staré/duplicitní kódy
- **Smazat Universal Analytics (UA):**
  ```html
  <script async src="https://www.googletagmanager.com/gtag/js?id=UA-114321355-1"></script>
  ```
  + všechny navazující `gtag('config','UA-114321355-1', …)` nebo `ga('create'…)` volání.

- **Smazat duplicity GA4**, pokud už jsou v šabloně:
  ```html
  <script async src="https://www.googletagmanager.com/gtag/js?id=G-BHQEJ5EC2D"></script>
  <script> gtag('js', new Date()); gtag('config','G-BHQEJ5EC2D'); </script>
  ```

### 2) Vložit obsah `cookie_consent_full.min.html`
- Část v `<head>` (preload + GA4) vložit do hlavičky před jakýkoli jiný Google kód.  
- Část v `<body>` (CSS/HTML/JS lišty) vložit na konec těla, nejlépe těsně nad `</body>`.

### 3) Ověřit pořadí skriptů
Správné pořadí v `<head>`:
1. `dataLayer`, funkce `gtag()`, `gtag('consent','default', …)`  
2. `gtag.js?id=G-BHQEJ5EC2D`  
3. `gtag('js', new Date()); gtag('config', 'G-BHQEJ5EC2D', {'wait_for_update':500})`

### 4) Odkaz na cookies stránku
- V banneru i modal okně odkazuje na: `https://mojra.cz/page/cookies`.  
- Pokud se URL změní, upravit přímo v HTML.

### 5) CSP (Content-Security-Policy), pokud je nasazena
Doplnit povolené zdroje:
- `script-src`: `https://www.googletagmanager.com https://www.google-analytics.com 'self'` (+ nonce/hash pro inline)
- `connect-src`: `https://www.google-analytics.com`
- `img-src`: `https://www.google-analytics.com data:`
- `style-src`: povolit inline `<style>`

### 6) Cache / verzování
Pokud extrahujete CSS/JS lišty do samostatných souborů, přidejte verze (hash/param).  
Po nasazení vyčistit cache.

---

## ✅ QA Checklist

1. Otevřít web v **Incognito** → banner se zobrazí, GA4 nic neposílá.  
2. Klik „Přijmout vše“ → zmizí banner, v `localStorage` se uloží `mojra_consent_v2`, GA4 začne posílat eventy.  
3. Klik „Odmítnout“ → GA4 neposílá data.  
4. Klik na kulaté tlačítko vlevo dole → otevře se modal, lze změnit volby.  
5. V konzoli: `dataLayer` obsahuje `consent/update`.  
6. Reset:  
   ```js
   localStorage.removeItem('mojra_consent_v2'); location.reload();
   ```

---

## 📌 Poznámky
- Nepoužíváme GTM (Google Tag Manager), běží přímo `gtag.js`.
- Žádné měření se nespustí bez souhlasu (kromě functionality & security storage = granted).
- Lišta ukládá jen do `localStorage`. Pro GDPR audit trail by bylo nutné doplnit ukládání na server (IP, timestamp, volba).

---

## 🔎 Diff kódu

**PRYČ:**
```diff
- <script async src="https://www.googletagmanager.com/gtag/js?id=UA-114321355-1"></script>
- gtag('config','UA-114321355-1', {...});
- <script async src="https://www.googletagmanager.com/gtag/js?id=G-BHQEJ5EC2D"></script>   (duplicitní verze)
- gtag('js', new Date()); gtag('config','G-BHQEJ5EC2D', {...}); (duplicitní verze)
```

**ZŮSTAT (z balíčku):**
```html
<!-- preload consent default (v head) -->
<script>window.dataLayer=window.dataLayer||[];function gtag(){dataLayer.push(arguments)};gtag('consent','default',{...});</script>

<!-- gtag.js GA4 + config s wait_for_update -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-BHQEJ5EC2D"></script>
<script>gtag('js', new Date()); gtag('config','G-BHQEJ5EC2D',{'wait_for_update':500});</script>

<!-- v body: CSS/HTML/JS cookie lišty -->
```
