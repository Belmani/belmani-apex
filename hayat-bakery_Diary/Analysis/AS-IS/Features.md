AS-IS Analysis — hayat-baeckerei.de

1. Stack Tecnico
Platform: Hostinger Website Builder (Zyro) — no-code drag & drop
Asset CDN: assets.zyrosite.com con Cloudflare image transforms (format=auto,w=375,fit=crop)
Background video: Pexels stock video via URL diretta — non un asset proprietario
Meta generator: Hostinger Website Builder — esposto pubblicamente nell'HTML head
Footer: "Erstellt & betreut durch VoiceHub" — quindi non è nemmeno la tipa originale, è un'agenzia AI/no-code
Punto critico: il sito è intrappolato in un builder proprietario. Non c'è codice sorgente da recuperare, non c'è DB, non c'è niente da "migrare" se non i contenuti testuali e le immagini.

2. Struttura & Navigazione
Pagine esistenti:
PaginaURLContenutoHome/Hero video, galleria, contatti, recensioniSüßspeisen/backerei-wiesbaden~30 prodotti con foto e prezziFrühstück & Klassiker/fruhstuck-wiesbaden-backereida scrapareHerzhafte Spezialitäten/afghanisches-essen-wiesbadenda scrapareGetränke/cafe-wiesbaden-getrankeda scrapareTorte anfragen/torte-bestellen-wiesbadenform + galleria torte (15 foto)Über uns/backerei-wiesbaden-cafeda scrapareKontakt/backerei-wiesbaden-kontaktWhatsApp, tel, indirizzo
Criticità navigazione:

Menu hamburger su mobile con sottomenu per Speisekarte → due tap per arrivare ai prodotti
URL keyword-stuffed (/afghanisches-essen-wiesbaden) — brutto, da ripulire nel nuovo sito
Nessuna ancora interna, ogni sezione è una pagina separata — anti-pattern rispetto al one-page richiesto


3. Analisi Contenuti
Fotografie
Buona notizia: il materiale fotografico è abbondante e di qualità discreta. Sulla pagina Süßspeisen si contano oltre 30 prodotti fotografati, la galleria torte ha almeno 15 scatti. Il problema è che ogni prodotto ha due immagini duplicate nello stesso markup (una per desktop 768px, una per mobile 375px) — gestite dal builder, non da media query CSS. Nel nuovo sito si risolve con srcset e un componente <Image> di Next.js.
Alcune foto sono stock generiche (il muffin, il donut, il croissant hanno nomi tipo fabiano_pimentel-muffin, araf60plus-donut — Pixabay/Pexels). Da sostituire con scatti reali o eliminare.
Testi
Scarsi ma recuperabili. La home è essenziale: tre frasi, qualche titolo. Il menu è dati strutturati (nome prodotto + prezzo + unità) — perfetto per popolare il DB PostgreSQL direttamente.
Prezzi & Menu — dati estratti (Süßspeisen)
Baklava voll Pistazien  → 3,50€ / 100g
Havuc                   → n.d.
Baklava Walnuss         → 3,00€ / 100g
Sekerpare               → 2,50€ / 100g
Sobiyet                 → 3,00€ / 100g
Kadayıf voll Pistazie   → 3,00€ / 100g
Kadayıf                 → 3,00€ / 100g
Kadayıf Rolle           → 3,50€ / 100g
Mini Baklava            → 3,00€ / 100g
Baklava Milch           → 3,00€ / 100g
Şirin Baklava           → 2,50€ / 100g (appare DUE VOLTE — probabile errore)
Baklava Klassisch       → 3,00€ / 100g
Halka                   → n.d.
Tulumba                 → 3,50€ / 100g (nota: "Sucuk Walnuss" ha immagine ma no prezzo visibile)
Sambal                  → 3,00€ / 100g
Jalebi                  → 2,50€ / 100g
Verschiedene Kekse      → 3,00€ / 100g
Asia Sweet Box          → 3,00€ pro Stück
Creme Rolle             → n.d. pro Stück
Karamell Trilece        → 4,00€ pro Stück
Schoko Trilece          → 4,00€ pro Stück
Brownie                 → 4,00€ pro Stück
Erdbeer Trilece         → 4,00€ pro Stück
Kiwi Trilece            → n.d.
Berliner                → 1,20€ pro Stück
Schoko Muffin           → 1,50€ pro Stück
Donut                   → 1,20€ / 1,00€ (ambiguo)
Kuchen                  → ab 4,00€
Schoko/Butter Croissant → n.d.
Torte:
Herzform  15cm → 20€ / 20cm → 30€ / 30cm → 45€
Rund      15cm → 20€ / 20cm → 25€ / 30cm → 45€
Anomalie trovate: Şirin Baklava duplicato, prezzi mancanti su ~6 voci, unità di misura inconsistenti (100g vs pro Stück non sempre indicate).
Recensioni
Tre stelle Google aggregate, testo generico. Il builder le mostra come widget statico — non aggiornato in tempo reale. Nel nuovo sito si può integrare l'embed Google oppure lasciare statico con refresh manuale.

4. SEO — Stato Attuale
Punti di forza:

Meta title e description presenti su ogni pagina
OG tags e Twitter card configurati
Canonical URL definiti

Criticità:

Title tag ancora con "Afghanische" — da aggiornare (req. 1 del cliente)
Keywords nelle meta: "Afghanische Bäckerei, hausgemachte Kuchen, gemütliches Café" — generiche
URL slug keyword-stuffed e semanticamente brutti
Nessuna sitemap.xml visibile
Nessun JSON-LD structured data (LocalBusiness, Menu, FoodEstablishment) — grave mancanza per un business locale su Google Maps
Alt tag delle immagini: tutti vuoti (alt: senza valore) — accessibilità e SEO zero
Performance: doppio src per ogni immagine nel markup invece di srcset → HTML pesante

Il nuovo sito con Next.js App Router e JSON-LD LocalBusiness + FoodEstablishment darà un boost SEO immediato e misurabile.

5. Performance & Accessibilità
Senza Lighthouse diretto posso inferire dal markup:

Video background da Pexels esterno, non ottimizzato, caricato in autoplay — killer per mobile su 4G
Doppia immagine per ogni prodotto nel DOM — peso HTML raddoppiato inutilmente
Nessun lazy loading esplicito sulle immagini del menu (gestito forse dal builder ma non controllabile)
Font e CSS del builder Zyro — non rimovibili, overhead fisso
Nessun Service Worker / PWA — zero offline capability, rilevante per un cliente che vuole mobile-first


6. Funzionalità Mancanti (gap vs requirements)
RequirementAS-ISGapMobile-firstResponsive ma non pensato mobileNavigation pattern sbagliatoOne-page scrollMulti-paginaArchitettura da rifareBottom nav / no hamburgerHamburger con sottomenuDa costruireWi-Fi inlineSolo sul foglietto fisicoNon presenteMultilinguaSolo tedesco4 lingue mancantiForm torteLink a WhatsApp/telefonoNessun form strutturatoOrdini onlineAssenteFase 2Carrello/PagamentoAssenteFase 3

7. Criticità Maggiori — Ranked
Critica 1 — Brand identity incoerente
Il sito dice "Afghanisch" in titoli, URL, meta, testi. Il cliente vuole "Turkish". Questo non è solo un find & replace: impatta SEO, Google My Business, le recensioni esistenti ("Die afghansichen Spezialitäten sind sehr empfehlenswert") e la coerenza dei nomi dei prodotti (Baklava, Kadayıf, Tulumba sono turchi — c'è già una contraddizione nell'AS-IS).
Critica 2 — Nessun form di contatto strutturato
"Torte anfragen" porta a... tre pulsanti: WhatsApp, telefono fisso, telefono mobile. Non c'è nessun form. Le richieste arrivano via chat WhatsApp non tracciabile. Questo è il primo quick win concreto che il cliente percepirà subito.
Critica 3 — Dati menu non strutturati
I prodotti sono HTML statico generato dal builder, non dati. Prezzi mancanti, duplicati, unità inconsistenti. Prima di sviluppare dobbiamo fare una sessione di data cleaning col cliente per validare tutto il catalogo.
Critica 4 — Performance mobile
Video background da CDN esterno + immagini doppie + no lazy loading = sito lento su mobile, esattamente dove il cliente vuole eccellere.
Critica 5 — Zero structured data
Nessun schema.org. Google non "capisce" che questo è un ristorante/pasticceria con orari, indirizzo, menu. È il motivo per cui non appare in Google Maps con rich result.

8. Proposta DEADLINE
Con il vincolo di non intaccare il PRIMARY_GOAL (microservizi + portfolio 2.0 a settembre), propongo questo:
Go-live Fase 1: entro il 15 luglio 2026 (~9 settimane)
Cosa include la Fase 1:

Sito one-page completo, mobile-first
Bottom navigation
Menu completo con tutti e quattro i gruppi
Multilingua DE/TR/EN (IT e FR rimandati — troppo per fase 1, si aggiungono in una patch)
Form "Torte anfragen" strutturato con email notification
Sezione Wi-Fi in-page
JSON-LD LocalBusiness + Menu per SEO
Deploy su Railway con dominio migrato

Cosa non include la Fase 1 (fase 2, da calendarizzare dopo settembre):

Ordini online
Carrello
Pagamenti
AI cake rendering
IT e FR

Motivazione del timing:
9 settimane a 4h/die tue + tempo Gabri è plausibile se lo sprint è disciplinato. Lascia 6-8 settimane libere prima di settembre per il focus microservizi. La fase 1 ha scope chiaro e nessuna dipendenza da backend complesso — è fondamentalmente un Next.js statico con Fastify leggero per il form torte e il menu dal DB.

Quando arriva Gabri, il primo esercizio concreto che proporrei è scrapare le pagine mancanti (Frühstück, Herzhafte, Getränke, Über uns, Kontakt) e costruire insieme lo schema PostgreSQL del catalogo prodotti. È un esercizio SQL reale su dati reali, perfetto per chi viene da un corso con MySQL.