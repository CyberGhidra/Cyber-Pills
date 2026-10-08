# Open-Source Intelligence (OSINT) Fundamentals
## Image OSINT — 1. Reverse Image Searching

Il **Reverse Image Searching** (ricerca inversa per immagini) è una metodologia OSINT fondamentale che consiste nell'utilizzare un'immagine (o il suo URL) come query di input invece di una parola chiave testuale. 

L'obiettivo è individuare:
* La fonte o il creatore originario dell'immagine.
* Versioni a risoluzione più elevata o non ritagliate.
* Altri profili, siti web o forum in cui la stessa immagine (o varianti simili) è stata pubblicata.
* Luoghi, monumenti o contesti geografici non dichiarati esplicitamente.

---

### Motori e Strumenti di Ricerca Inversa

* **[Google Images](https://images.google.com)**
  * **Caratteristiche:** Integrato con la tecnologia **Google Lens**, eccelle nell'identificare oggetti, monumenti, prodotti commerciali, loghi e porzioni specifiche ritagliate all'interno di una scena complessa.
  * **Funzionamento:** Permette il caricamento diretto del file o l'inserimento dell'URL dell'immagine, offrendo una selezione dinamica dell'area di interesse (*crop-to-search*).

* **[Yandex Images](https://yandex.com/images/)**
  * **Caratteristiche:** Riconosciuto nella comunità OSINT come uno dei motori più potenti per il **riconoscimento facciale e biometrico** e per la corrispondenza visiva di sfondi, architetture e dettagli secondari.
  * **Punto di forza:** A differenza di altri motori occidentali che applicano filtri restrittivi sulla privacy dei volti, Yandex è in grado di rintracciare la stessa persona anche se fotografata con angolazioni, illuminazione o tagli di capelli differenti.

* **[TinEye](https://tineye.com)**
  * **Caratteristiche:** Motore specializzato nella corrispondenza esatta (*exact/modified match*) basato su impronte digitali percettive delle immagini (perceptual hashing).
  * **Punto di forza:** Ottimo per determinare la cronologia temporale: consente di ordinare i risultati per **"Oldest"** (la versione più vecchia indicizzata nel web) per risalire alla fonte primaria o all'autore originale.

---

### Esercitazioni e Materiale di Studio

* **Cartella Risorse & Challenge:**
  * [Course photos + challenges (Google Drive)](https://drive.google.com/drive/folders/1xMADgUUoJ0A-plnoFzcHY7rBh7-3ZYTT?usp=sharing)
  * Include le immagini di test e gli scenari pratici analizzati nel corso per sperimentare le differenze di indicizzazione tra i diversi motori.

---

### Metodologia Operativa Consigliata

1. **Test Multi-Motore:** Non limitarsi mai a un solo servizio: un'immagine non indicizzata da Google può produrre riscontri immediati su Yandex o TinEye, e viceversa.
2. **Ritagli Mirati (Cropping):** Se la ricerca dell'immagine intera non dà risultati, ritagliare singoli elementi distintivi sullo sfondo (insegne, monumenti, veicoli con targhe visibili, vestiti particolari).
3. **Controllo Speculare (Flips/Rotazioni):** Spesso le immagini ripubblicate o usate come esche/truffa vengono ribaltate orizzontalmente per eludere gli algoritmi di confronto automatico; ribaltare l'immagine e ripetere la scansione.


## Image OSINT — 2. Viewing EXIF Data & Geolocation

I dati **EXIF** (*Exchangeable Image File Format*) sono metadati incorporati automaticamente dai dispositivi digitali (smartphone, reflex, droni) al momento dello scatto di una fotografia o registrazione di un video. 

Nell'ambito delle indagini OSINT, l'analisi EXIF rappresenta una delle fonti primarie per la verifica delle prove, l'attribuzione temporale e la **geolocalizzazione fisica** del target.

---

### Metadati Rilevabili tramite EXIF

* **Coordinate Geografiche (GPS):** Latitudine, longitudine e altitudine esatte del punto di scatto.
* **Timestamp e Datazione:** Data e ora precisa di creazione e modifica del file (fondamentale per stabilire una timeline degli eventi).
* **Specifiche del Dispositivo:** Marca, modello esatto di smartphone/fotocamera, numero di serie hardware, versione del firmware/sistema operativo.
* **Parametri Fotografici:** Lunghezza focale, apertura del diaframma, tempo di esposizione, sensibilità ISO, orientamento e software di post-produzione/fotoritocco utilizzato.

> **Nota Operativa:** La maggior parte dei social network moderni (es. Twitter/X, Instagram, Facebook) elimina automaticamente i metadati EXIF durante il caricamento per proteggere la privacy degli utenti. Tuttavia, immagini scaricate da blog, siti web, archivi cloud, forum, messaggistica istantanea non compressa o repository conservano spesso i metadati intatti.

---

### Strumenti per la Lettura EXIF

* **[Jimpl](https://jimpl.com/)**
  * Servizio web rapido per caricare un'immagine, visualizzarne ed estrarne tutti i metadati EXIF nascosti (inclusi i tag GPS visualizzati su mappa integrata), con opzione per rimuovere i metadati a scopo di OPSEC difensiva.
* **Strumenti da Riga di Comando (CLI):**
  * `exiftool`: Lo strumento standard di riferimento in ambiente Linux/macOS per l'analisi forense avanzata e l'estrazione esaustiva di metadati da qualsiasi file multimediale.

---

### Geolocalizzazione e Tecniche di Indagine Ambientale

Quando i metadati GPS sono assenti o sono stati rimossi, la geolocalizzazione si sposta sull'analisi degli indizi visivi all'interno della scena:

* **[Google Maps & Google Street View](https://www.google.com/maps)**
  * Piattaforma fondamentale per verificare a livello del suolo (*ground-truth verification*) edifici, skyline, incroci stradali, conformazione del terreno e punti di riferimento.
* **[GeoGuessr](https://www.geoguessr.com)**
  * Piattaforma di allenamento ludico basata su Street View, ampiamente impiegata da analisti OSINT per affinare le capacità di deduzione visiva rapida.
* **[GeoGuessr - The Top Tips, Tricks and Techniques](https://somerandomstuff1.wordpress.com/2019/02/08/geoguessr-the-top-tips-tricks-and-techniques/)**
  * Guida metodologica completa ai dettagli per dedurre il paese o la regione geografica:
    * **Segnaletica stradale:** Colore delle strisce di mezzeria (gialle vs bianche), cartelli di dare la precedenza, limiti di velocità, lingue/alfabeti (cirillico, arabo, kanji/katakana).
    * **Infrastrutture urbane:** Tipologia dei pali elettrici (in legno, cemento forato), guard-rail, cassette postali, targhe automobilistiche (banda blu europea, formato lungo vs quadrato).
    * **Paesaggio e vegetazione:** Specie di alberi, clima, tipo di suolo, posizione del sole (emisfero nord vs emisfero sud per dedurre i punti cardinali).
    * **Architettura:** Stile delle coperture dei tetti, materiali di costruzione e antenne paraboliche.

---

### Materiale Didattico del Corso

* **Cartella Risorse & Esercizi Pratici:**
  * [Course photos + challenges (Google Drive)](https://drive.google.com/drive/folders/1xMADgUUoJ0A-plnoFzcHY7rBh7-3ZYTT?usp=sharing)
  * Contiene le immagini impiegate durante le lezioni per esercitarsi nell'estrazione dei metadati GPS tramite Jimpl e nella successiva validazione della posizione su Google Maps.
