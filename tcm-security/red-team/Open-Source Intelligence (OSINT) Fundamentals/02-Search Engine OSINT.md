# Open-Source Intelligence (OSINT) Fundamentals
## 2. Search Engine Operators (Google Dorking & Search Syntax)

I **Search Engine Operators** (operatori di ricerca avanzata, noti in ambito di sicurezza anche come *Google Dorks*) sono comandi sintattici che consentono di filtrare, isolare e interrogare in modo mirato gli indici dei motori di ricerca. 

Nel contesto OSINT, queste query servono a scovare directory esposte, file di configurazione, credenziali trapelate, documenti indicizzati per errore o profili specifici all'interno di social network.

---

### Motori di Ricerca e Risorse di Riferimento

* **Google & Google Advanced Search:**
  * [Google Search](https://www.google.com/) — Motore primario per indicizzazione web e volume di dati.
  * [Google Advanced Search](https://www.google.com/advanced_search) — Interfaccia grafica guidata per comporre query complesse senza ricordare tutti gli operatori.
  * [Google Search Guide (PDF)](http://www.googleguide.com/print/adv_op_ref.pdf) — Riferimento sintetico ufficiale sugli operatori avanzati e la loro sintassi.
* **Bing:**
  * [Bing](https://www.bing.com/) — Motore di ricerca alternativo, spesso utile per trovare risorse o IP non indicizzati da Google.
  * [Bing Search Guide (Bruce Clay)](https://www.bruceclay.com/blog/bing-google-advanced-search-operators/) — Guida comparativa dettagliata tra gli operatori supportati da Bing e quelli di Google.
* **Yandex & Baidu (Rilevanza Regionale):**
  * [Yandex](https://yandex.com/) — Motore di ricerca russo; eccelle in modo particolare nella ricerca per immagini (*Reverse Image Search*) e nel parsing di dati in Europa orientale/area russofona.
  * [Baidu](http://www.baidu.com/) — Il motore di ricerca dominante in Cina, indispensabile per investigazioni OSINT su target, aziende o servizi dell'area asiatica.
* **DuckDuckGo:**
  * [DuckDuckGo](https://duckduckgo.com/) — Motore incentrato sulla privacy che supporta sia operatori classici sia scorciatoie veloci (*Bangs*).
  * [DuckDuckGo Search Syntax Guide](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/) — Manuale ufficiale sulla sintassi e sui filtri supportati da DuckDuckGo.

---

### Operatori di Ricerca Principali

| Operatore | Scopo | Esempio Pratico |
| :--- | :--- | :--- |
| `site:` | Limita i risultati a un dominio, sottodominio o TLD specifico. | `site:twitter.com` oppure `site:.gov.it` |
| `inurl:` | Cerca pagine che contengono la parola specificata all'interno del percorso URL. | `inurl:admin` o `inurl:login` |
| `allinurl:` | Richiede che **tutte** le parole indicate siano presenti nell'URL. | `allinurl:wp-content uploads` |
| `intitle:` | Cerca termini presenti nel tag `<title>` della pagina HTML. | `intitle:"index of /"` |
| `allintitle:` | Richiede che tutti i termini compaiano nel titolo della pagina. | `allintitle:dashboard restricted` |
| `filetype:` / `ext:` | Filtra i file per estensione (es. PDF, DOCX, XLSX, SQL, TXT, LOG). | `filetype:pdf` o `filetype:env` |
| `intext:` / `allintext:` | Cerca termini esclusivamente all'interno del corpo del testo della pagina. | `intext:"confidential"` |
| `" "` (Virgolette) | Ricerca esatta per stringa/frase, preservando ordine e punteggiatura. | `"password reset token"` |
| `-` (Meno / Esclusione) | Esclude termini, siti o estensioni dai risultati. | `site:target.com -www` |
| `OR` / `\|` | Operatore logico booleano per unire due criteri alternativi. | `filetype:pdf OR filetype:docx` |
| `*` (Wildcard) | Segnaposto che sostituisce una o più parole sconosciute. | `"username * password"` |

---

### Analisi del Caso Pratico: `inurl:password site:twitter.com`

La query citata nell'esempio:

```text
inurl:password site:twitter.com
```
site:twitter.com: Vincola il crawler a mostrare unicamente risultati provenienti dal dominio twitter.com (o x.com).

```text
inurl:password:
```
Cerca pagine in cui la stringa password compare direttamente nell'URL (ad esempio pagine di reset credenziali, endpoint di autenticazione o tweet/post contenenti link con percorsi specifici).

Applicazioni OSINT di combinazioni simili:
Ricerca di credenziali o file sensibili esposti:

```text
Plaintext
site:target.com filetype:log intext:password
site:target.com filetype:env "DB_PASSWORD"
```

Individuazione di Directory Listing non protetti:

```text
Plaintext
site:target.com intitle:"index of /" "backup"
```
Esplorazione di account o conversazioni social mirate:
```text
Plaintext
site:twitter.com intext:"api key" "pastebin"
```
Differenze Operative tra Motori di Ricerca
Google: Ha il più ampio database indicizzato e la sintassi più potente (site:, inurl:, filetype:, intitle:), ma applica rigidi controlli anti-bot (CAPTCHA frequenti) quando rileva ricerche con dorking intensivo.

Bing: Supporta filtri come contains: per rintracciare pagine con specifici tipi di file incorporati e ip: per trovare host sullo stesso indirizzo IP.

DuckDuckGo: Accetta filtri come site:, filetype:, ma è particolarmente utile per le sue scorciatoie !bang (es. !w per Wikipedia, !gh per GitHub).

Yandex: Permette operatori avanzati (es. rhost:, mime:) e spesso indicizza risorse escluse dalle blacklist occidentali.
