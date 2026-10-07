# 7. Dictionaries (Dizionari)

I dizionari in Python sono collezioni mutabili di elementi strutturati in coppie chiave-valore (`key: value`), racchiusi tra parentesi graffe `{}` o inizializzati tramite il costruttore `dict()`. Consentono la ricerca ad accesso rapido per chiave e supportano metodi integrati per estrarre chiavi, valori, coppie o aggiornare i dati in modo dinamico.

---

### Codice Sorgente Completo (`dict-demo.py`)

```python
# 1. Definizione e Ispezione
dict1 = {"a": 1, "b": 2, "c": 3}
print(dict1)                         # {'a': 1, 'b': 2, 'c': 3}
print(type(dict1))                   # <class 'dict'>
print(len(dict1))                    # 3

# 2. Accesso ai Valori
print(dict1["a"])                    # 1 (Accesso diretto tramite parentesi quadre)
print(dict1.get("a"))                # 1 (Accesso sicuro tramite metodo .get())

# 3. Viste del Dizionario (Keys, Values, Items)
print(dict1.keys())                  # dict_keys(['a', 'b', 'c'])
print(dict1.values())                # dict_values([1, 2, 3])
print(dict1.items())                 # dict_items([('a', 1), ('b', 2), ('c', 3)])

# 4. Modifica e Aggiunta di Elementi
dict1["a"] = 1                       # Riassegnazione del valore per la chiave esistente "a"
print(dict1)                         # {'a': 1, 'b': 2, 'c': 3}

dict1["d"] = 4                       # Inserimento di una nuova coppia chiave-valore ("d": 4)
print(dict1)                         # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

dict1["a"] = 0                       # Aggiornamento del valore di "a" a 0
print(dict1)                         # {'a': 0, 'b': 2, 'c': 3, 'd': 4}

dict1.update({"a": 1})               # Aggiornamento tramite metodo .update()
print(dict1)                         # {'a': 1, 'b': 2, 'c': 3, 'd': 4}

# 5. Rimozione di Elementi
dict1.pop("d")                       # Rimuove la chiave "d" e ne restituisce il valore
print(dict1)                         # {'a': 1, 'b': 2, 'c': 3}

del dict1["c"]                       # Eliminazione della chiave "c" con l'istruzione 'del'
print(dict1)                         # {'a': 1, 'b': 2}

# 6. Annidamento (Nested Dictionaries)
dict1["c"] = {"a": 1, "b": 2}        # Inserimento di un dizionario come valore
print(dict1)                         # {'a': 1, 'b': 2, 'c': {'a': 1, 'b': 2}}

# 7. Inizializzazione di Dizionari Vuoti
dict2 = {}                           # Dizionario vuoto tramite parentesi graffe
print(dict2)                         # {}

dict3 = dict()                       # Dizionario vuoto tramite costruttore dict()
print(dict3)                         # {}

```
# 8. Sets (Insiemi)

I set in Python sono collezioni mutabili di elementi univoci e non ordinati. Si definiscono racchiudendo elementi delimitati da virgole tra parentesi graffe `{}` oppure usando il costruttore `set()`. Rimuovono automaticamente i duplicati e supportano operazioni insiemistiche come unione, aggiunta e rimozione.

---

### Codice Sorgente Completo (`sets-demo.py`)

```python
# 1. Definizione e Ispezione del Tipo
set1 = {"a", "b", "c"}
print(set1)                         # {'c', 'b', 'a'} (l'ordine degli elementi non è garantito)
print(type(set1))                   # <class 'set'>

# 2. Eliminazione Automatica dei Duplicati
set2 = {"a", "a", "a"}
print(set2)                         # {'a'}
print(len(set2))                    # 1

# 3. Tipi Misti e Equivalenza Booleana/Numerica (0 == False, 1 == True)
set3 = {"a", 0, True}
print(set3)                         # {0, True, 'a'}

# Conversione di una sequenza (tupla) in set con set()
set4 = set(("b", 1, False))
print(set4)                         # {False, 1, 'b'}

# 4. Aggiunta di Elementi Singoli e Multipli
set1.add("d")                       # Inserisce un singolo elemento con .add()
print(set1)                         # {'c', 'a', 'b', 'd'}

set3.update(set4)                   # Unisce gli elementi di un altro set con .update()
print(set3)                         # {0, True, 'b', 'a'} (0 equivale a False, 1 a True: nessun duplicato)

# Aggiornamento da una Lista
list1 = ["a", "b", "c"]
set4 = {4, 5, 6}
print(list1)                        # ['a', 'b', 'c']
print(set4)                         # {4, 5, 6}

set4.update(list1)                  # .update() accetta qualsiasi iterabile (es. list)
print(set4)                         # {4, 5, 6, 'c', 'b', 'a'}

# 5. Operazione di Unione
set5 = {4, 5, 6}
set6 = set4.union(set4)             # .union() restituisce un nuovo set unendo gli elementi
print(set6)                         # {4, 5, 6, 'c', 'a', 'b'}

# 6. Rimozione di Elementi (.discard vs .pop)
set4.discard(4)                     # Rimuove l'elemento 4 senza sollevare errori se non esiste
print(set4)                         # {5, 6, 'c', 'a', 'b'}

set4.discard(4)                     # Nessuna eccezione se l'elemento è già assente
print(set4)                         # {5, 6, 'c', 'a', 'b'}

print(set1)                         # {'a', 'd', 'c', 'b'}
set1.pop()                          # Rimuove ed estrae un elemento arbitrario
print(set1)                         # {'d', 'c', 'b'}

```
# 9. Conditionals (Condizionali)

Le istruzioni condizionali in Python consentono di eseguire blocchi di codice specifici in base alla valutazione di espressioni booleane (`True` o `False`). Utilizzano la struttura principale composta da `if`, clausole intermedie opzionali `elif` (else if) e una clausola finale `else`. Python supporta anche espressioni condizionali inline su singola riga (operatore ternario).

---

### Codice Sorgente Completo (`conditional-demo.py`)

```python
# 1. Valutazione Diretta di Valori Booleani e Negazione
if True:
    print("True")                   # Eseguito: la condizione è direttamente True

if False:
    print("False")                  # Non eseguito: la condizione è False

if not False:
    print("not false")              # Eseguito: not inverte False in True

# 2. Struttura if / elif / else con Operatori di Confronto
if 1 < 1:
    print("1 < 1")                  # Falso: 1 non è strettamente minore di 1
elif 1 <= 1:
    print("1 <= 1")                 # Vero: 1 è minore o uguale a 1 (blocco eseguito)
else:
    print("else 1")                 # Non valutato: un ramo precedente è già risultato True

# Cascata di rami elif
if 1 < 1:
    print("1 < 1")
elif 1 < 1:
    print("1 <= 1")
elif 2 < 2:
    print("2 <= 2")

# 3. Operatori Logici Composti (and, or)
if 1 > 0 and 0 < 1:
    print("1 > 0 and 0 < 1")        # Eseguito: entrambe le condizioni sono vere

if 1 > 0 and 0 < 1:
    print("1 > 0 and 0 < 1")        # Eseguito: duplicato nel codice originale

# Composizione di logiche multiple con parentesi e or
if (1 < 0 or 0 < 1) or 1 == 0:
    print("1 < 0 or 0 < 1")         # Eseguito: la sotto-condizione (0 < 1) rende vero l'intero blocco

# 4. Istruzioni Condizionali Inline (One-liner / Operatore Ternario)
if 0 < 1: print("0 < 1")            # If su singola riga per istruzioni semplici

# Operatore ternario: [azione se vero] if [condizione] else [azione se falso]
print("1 >= 1") if 1 >= 1 else print("1 < 1")

# Struttura classica equivalente all'operatore ternario sopra
if 1 >= 1:
    print("1 >= 1")
else:
    print("1 < 1")

# 5. Valutazione Sequenziale con Fallback su else
if 0 < 0:
    print("1")
elif 0 > 1:
    print("2")
else:
    print("3")                      # Eseguito: tutte le condizioni precedenti sono risultate False

# Operatore ternario annidato (condizionale a catena su singola riga)
print("1") if 0 < 0 else print("2") if 0 > 1 else print("3")
```
# 10. Loops (Cicli)

I cicli in Python consentono di ripetere l'esecuzione di blocchi di codice. Il ciclo `while` continua a iterare finché una data condizione booleana rimane vera, mentre il ciclo `for` itera sequenzialmente sugli elementi di un qualsiasi oggetto iterabile (liste, range, stringhe, dizionari). Python supporta cicli annidati e istruzioni di controllo del flusso come `break`, `continue` e `pass`.

---

### Codice Sorgente Completo (`loops-demo.py`)

```python
# 1. Approccio Sequenziale vs Ciclo While
a = 1
print(a)                             # 1
a += 1
print(a)                             # 2
a += 1
print(a)                             # 3
a += 1
print(a)                             # 4
a += 1
print(a)                             # 5

# Ciclo While: ripete finché la condizione a < 5 è verificata
a = 1
while a < 5:
    a += 1
    print(a)                         # Stampa: 2, 3, 4, 5

# 2. Ciclo For con Liste e Funzione range()
# Iterazione su una lista esplicita
for i in [0, 1, 2, 3, 4]:
    print(i + 6)                     # Stampa: 6, 7, 8, 9, 10

print("---")

# Iterazione equivalente tramite range(5) (genera interi da 0 a 4)
for i in range(5):
    print(i + 6)                     # Stampa: 6, 7, 8, 9, 10

# 3. Cicli Annidati (Nested Loops)
for i in range(3):
    for j in range(3):
        print(i, j)                  # Genera le coppie: (0,0), (0,1), (0,2), (1,0)... (2,2)

print("---")

# 4. Istruzione break (Interruzione del Ciclo)
for i in range(5):
    if i == 2:
        break                        # Termina ed esce immediatamente dal ciclo
    print(i)                         # Stampa solo: 0, 1

print("---")

# 5. Istruzione continue (Salto dell'Iterazione Corrente)
for i in range(5):
    if i == 2:
        continue                     # Salta il resto del blocco e passa all'iterazione successiva
    print(i)                         # Stampa: 0, 1, 3, 4 (il 2 viene ignorato)

print("---")

# 6. Istruzione pass (Segnaposto Null-Op)
for i in range(5):
    if i == 2:
        pass                         # Non compie alcuna azione; l'esecuzione prosegue normalmente
    print(i)                         # Stampa: 0, 1, 2, 3, 4

# 7. Iterazione su Stringhe e Viste di Dizionari
for c in "string":
    print(c)                         # Itera carattere per carattere: 's', 't', 'r', 'i', 'n', 'g'

for key, value in {"a": 1, "b": 2, "c": 3}.items():
    print(key, value)                # Unpacking delle tuple (chiave, valore): a 1, b 2, c 3
```
# 11. Reading and Writing Files (Lettura e Scrittura File)

La gestione dei file in Python avviene tramite la funzione integrata `open()`. Essa consente di specificare la modalità di apertura (`'r'` per lettura, `'w'` per scrittura, `'a'` per append, `'t'` per testo) e la codifica dei caratteri (`encoding`). È buona prassi gestire il puntatore del file tramite `.seek()`, chiudere le risorse con `.close()` o utilizzare il context manager `with open(...)` per la chiusura automatica e sicura dei descrittori di file.

---

### Codice Sorgente Completo (`files-demo.py`)

```python
# 1. Apertura e Ispezione dell'Oggetto File
f = open('top-100.txt')
print(f)                             # <_io.TextIOWrapper name='top-100.txt' mode='r' encoding='UTF-8'>

# Apertura esplicita in modalità lettura testo ('rt')
f = open('top-100.txt', 'rt')
print(f)

# 2. Lettura con readlines() e Gestione del Cursore con seek()
print(f.readlines())                 # Legge l'intero file e restituisce una lista di righe
print(f.readlines())                 # Restituisce [] perché il cursore si trova alla fine del file (EOF)

f.seek(0)                            # Riposiziona il cursore all'inizio del file (offset 0)
print(f.readlines())                 # Rilegge nuovamente tutte le righe

# 3. Iterazione Riga per Riga e Pulizia dell'A Capo
f.seek(0)
for line in f:
    print(line.strip())              # line.strip() rimuove gli spazi bianchi e il carattere finale '\n'

f.close()                            # Chiusura manuale del file

# 4. Scrittura in Modalità Append ('a')
f = open("test.txt", "a")
f.write("test line two!")            # Accoda la stringa in fondo al file esistente
f.close()

# 5. Attributi dell'Oggetto File
print(f.name)                        # 'test.txt' (nome del file)
print(f.closed)                      # True (verifica se il descrittore è stato chiuso)
print(f.mode)                        # 'a' (modalità con cui era stato aperto)

# 6. Context Manager 'with open' e Gestione delle Codifiche (Encoding)
# Caso di errore tipico su file dizionario/wordlist (es. rockyou.txt in UTF-8):
# with open('rockyou.txt') as f:
#     for line in f:
#         print(line.strip())
# -> Solleva: UnicodeDecodeError: 'utf-8' codec can't decode byte ...

# Risoluzione specificando la corretta codifica (es. 'latin-1'):
with open('rockyou.txt', encoding='latin-1') as f:
    for line in f:
        print(line.strip())          # Lettura sicura senza errori di decodifica
 ```       
