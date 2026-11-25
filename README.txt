VÂND RAPID - prima versiune Android (PWA)

Conținut:
- index.html: aplicația
- manifest.webmanifest: configurare instalare
- sw.js: cache offline de bază

Funcții:
- adăugare/editare produs, fotografie, preț, stare și descriere
- salvare locală în browser (localStorage)
- catalog și anunț pregătit
- copiere text și distribuire prin meniul Android/WhatsApp
- deschidere Facebook Marketplace și TikTok Upload

Instalare pe Android:
1. Publică folderul pe un hosting HTTPS (de exemplu GitHub Pages sau un hosting web propriu).
2. Deschide adresa site-ului în Chrome pe telefon.
3. Apasă meniul ⋮ > „Instalează aplicația” sau „Adaugă la ecranul de pornire”.
4. Aplicația va apărea ca Vând Rapid.

Limitări importante:
- Acesta este un PWA instalabil pe Android, nu un APK nativ.
- Nu publică automat în conturi. Facebook/TikTok pot cere pași manuali; automatizarea necesită API-uri oficiale, conturi eligibile și aprobări.
- Datele sunt stocate local în browserul dispozitivului. Ștergerea datelor browserului poate șterge catalogul.
- Fotografiile sunt păstrate local; pentru multe fotografii, spațiul browserului poate deveni limitat.
- Nu include încă sincronizare în cloud, autentificare, backup sau generare AI de descrieri.

Pentru o versiune APK nativă este necesar un mediu Android SDK/Gradle și semnarea pachetului.
