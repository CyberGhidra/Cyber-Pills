# 12. User Input (Input dell'Utente)

La funzione built-in `input()` permette di acquisire dati inseriti da tastiera tramite riga di comando. Il programma sospende l'esecuzione in attesa dell'invio (`Enter`), restituendo sempre il dato digitato sotto forma di stringa (`str`). Può essere impiegata sia per acquisizioni singole sia all'interno di cicli per implementare prompt interattivi o condizioni di uscita.

---

### Codice Sorgente Completo (`input-demo.py`)

```python
# 1. Acquisizione Semplice senza Messaggio
test = input()
print(test)

# 2. Acquisizione con Messaggio di Prompt
test = input("Enter the IP:")
print(test)

# 3. Prompt Interattivo Continuo con Condizione di Uscita (Loop while)
while True:
    test = input("Enter the IP: ")
    print(">>> {}".format(test))
    if test == "exit":
        break
    else:
        print("exploiting..")
```
# 13. Exceptions and Error Handling (Gestione delle Eccezioni ed Errori)

La gestione delle eccezioni in Python permette di intercettare e gestire errori durante il tempo di esecuzione (runtime) evitando che il programma si arresti improvvisamente. Si basa sui blocchi `try` ed `except`, a cui si possono associare tipi di eccezioni specifiche (es. `FileNotFoundError`), clausole di pulizia finale con `finally`, la generazione esplicita di errori tramite `raise` e controlli di integrità con l'istruzione `assert`.

---

### Codice Sorgente Completo (`exceptions-demo.py`)

```python
# 1. Esecuzione Lineare Semplice
print(1)
print(2)

# 2. Blocco try / except Generico
try:
    fdglkjgkldfjgd                   # Variabile o istruzione non definita (NameError)
except:
    print("the file does not exist!")

# 3. Blocco try / except Specifico, Fallback e Clausola finally
try:
    f = open("top-100.txt")          # Tentativo di apertura file
except FileNotFoundError:
    print("the file does not exist!") # Intercetta l'assenza del file
except Exception as e:
    print(e)                         # Intercetta qualsiasi altra eccezione generica salvandola in 'e'
finally:
    print("this message!")           # Eseguito SEMPRE, sia in caso di successo che di errore

# 4. Sollevamento Esplicito di Eccezioni con raise
n = 100
if n == 0:
    raise Exception("n can't be 0!")
if type(n) is not int:
    raise Exception("n must be an int!")

print(1 / n)                         # Stampa: 0.01

# 5. Controllo delle Precondizioni con assert
n = 0
assert (n != 0)                      # Se la condizione è False, solleva AssertionError
print(1 / n)
```
# 14. Comprehensions (List e Set Comprehensions)

Le *comprehension* in Python offrono una sintassi compatta, dichiarativa ed efficiente per costruire nuove collezioni (come liste o set) a partire da sequenze o iterabili esistenti. Consentono di applicare trasformazioni, filtri condizionali (`if`), ramificazioni con operatore ternario (`if-else`) e appiattimenti di liste multidimensionali su una sola riga, risultando più leggibili ed espressive rispetto ai tradizionali cicli `for` imperativi con `.append()`.

---

### Codice Sorgente Completo (`comprehension-demo.py`)

```python
# 1. Definizione di Base e Clonazione con List Comprehension
list1 = ['a', 'b', 'c']
print(list1)

# Crea una nuova lista iterando su ciascun elemento di list1
list2 = [x for x in list1]
print(list2)

# 2. Filtraggio Condizionale (Clausola 'if' in Coda)
list3 = [x for x in list1 if x == 'a']
print(list3)                         # ['a']

# 3. Generazione e Trasformazione tramite range() e hex()
list4 = [x for x in range(5)]
print(list4)                         # [0, 1, 2, 3, 4]

list5 = [hex(x) for x in range(5)]
print(list5)                         # ['0x0', '0x1', '0x2', '0x3', '0x4']

# 4. Condizionale con Operatore Ternario (Clausola 'if-else' all'Inizio)
# Se x > 0 converte in esadecimale, altrimenti imposta la stringa "X"
list6 = [hex(x) if x > 0 else "X" for x in range(5)]
print(list6)                         # ['X', '0x1', '0x2', '0x3', '0x4']

# 5. Operazioni Aritmetiche e Condizioni Multiple (or)
list7 = [x * x for x in range(5)]
print(list7)                         # [0, 1, 4, 9, 16]

list8 = [x for x in range(5) if x == 0 or x == 1]
print(list8)                         # [0, 1]

# 6. Liste Matrice (Bidimensionali) e Appiattimento (Flattening)
list9 = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print(list9)                         # [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

# Cicli for annidati: per ogni sottolista x in list9, per ogni elemento y in x
list10 = [y for x in list9 for y in x]
print(list10)                        # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# 7. Set Comprehension
# Usa le parentesi graffe {} per generare automaticamente un set
set1 = {x + x for x in range(5)}
print(set1)                          # {0, 2, 4, 6, 8}

# 8. Comprehension su Stringhe e Ricostruzione con .join()
list11 = [c for c in "string"]
print(list11)                        # ['s', 't', 'r', 'i', 'n', 'g']

print("".join(list11))               # Ricostruzione: 'string'
print("-".join(list11))              # Unione con delimitatore: 's-t-r-i-n-g'

# 9. Confronto con l'Approccio Imperativo Tradizionale
list12 = []
for c in "string":
    list12.append(c)
print(list12)                        # ['s', 't', 'r', 'i', 'n', 'g']
```
# 15. Functions and Code Reuse (Funzioni e Riuso del Codice)

Le funzioni in Python consentono di organizzare il codice in blocchi logici riutilizzabili, modulari e manutenibili. Vengono definite tramite la parola chiave `def` e possono restituire valori con `return`, accettare parametri posizionali o nominali (keyword arguments), definire valori di default, gestire un numero arbitrario di argomenti posizionali (`*args`) o con nome (`**kwargs`), interagire con lo scope globale tramite `global` e implementare la ricorsione[cite: 5, 6, 7, 8].

---

### Codice Sorgente Completo (`functions-demo.py`)

```python
# 1. Definizione e Chiamata di Base
def function1():
    print("hello from function!")

function1()
function1()

# 2. Valori di Ritorno con return
def function2():
    return "hello from function2!"

return_from_function2 = function2()
print(return_from_function2)

# 3. Parametri Singoli e Formattazione
def function3(s):
    print("\t{}".format(s))

function3("parameter")
function3("parameter2")

# 4. Parametri Multipli: Posizionali vs Nominali (Keyword Arguments)
def function4(s1, s2):
    print("{} {}".format(s1, s2))

function4("any", "thing")                          # Argomenti posizionali: 'any thing'
function4(s1="thing", s2="any")                   # Argomenti nominali espliciti: 'thing any'
function4(s2="any", s1="thing")                   # L'ordine non conta se specificati per nome: 'thing any'

# 5. Argomenti con Valori Predefiniti (Default Arguments)
def function5(s1="default"):
    print("{}".format(s1))

function5()                                       # Usa il valore di default: 'default'
function5("anything")                             # Sovrascrive il default: 'anything'

# 6. Argomenti Posizionali Arbitrari (*args / *more)
def function6(s1, *more):
    print("{} {}".format(s1, " ".join([s for s in more])))

function6("function6")
function6("function6", "a")
function6("function6", "a", "b", "c")

# 7. Argomenti Nominali Arbitrari (**kwargs / **ks)
def function7(**ks):
    for a in ks:
        print(a, ks[a])

function7(a="1", b="2", c="3", d="4")

# 8. Tipizzazione Dinamica dei Parametri
def function8(s, f, i, l):
    print(type(s))
    print(type(f))
    print(type(i))
    print(type(l))

function8("string", 1.0, 1, ['l', 'i', 's', 't'])

# 9. Variabili Globali e Modifica dello Scope con global
v = 100
print(v)                                          # 100

def function9():
    global v
    v += 1
    print(v)

function9()                                       # 101
print(v)                                          # 101 (il valore globale è stato modificato)

# 10. Chiamate a Funzioni Annidate
def function10():
    print("hello from function10")

def function11():
    function10()
    print("hello from function11")

function11()

# 11. Ricorsione (Funzioni che chiamano se stesse)
def function12(x):
    print(x)
    if x > 0:
        function12(x - 1)

function12(5)                                     # Stampa: 5, 4, 3, 2, 1, 0

# 12. Confronto: Ricorsione vs Ciclo Iterativo Equivalente
def function13(x):
    while x >= 0:
        print(x)
        x -= 1

function13(5)                                     # Stampa: 5, 4, 3, 2, 1, 0
```
# 16. Lambdas (Funzioni Anonime Lambda)

In Python le espressioni `lambda` consentono di creare piccole funzioni anonime (prive di nome) su una singola riga. A differenza delle normali definizioni con `def`, le funzioni lambda contengono un'unica espressione il cui risultato viene restituito implicitamente senza la parola chiave `return`. Possono essere assegnate a variabili, invocate sul posto come Immediately Invoked Function Expressions (IIFE), impiegate per verifiche logiche veloci o combinate con le *list comprehension* per l'elaborazione e la trasformazione dei dati.

---

### Codice Sorgente Completo (`lambda-demo.py`)

```python
# 1. Definizione Base e Assegnazione a Variabile
# Sintassi: lambda <parametri>: <espressione_restituita>
add = lambda x, y: x + y
print(add(10, 4))                    # 14

# Equivalente tradizionale con 'def'
def addf(x, y):
    return x + y

print(addf(10, 4))                   # 14

# 2. Invocazione Immediata sul Posto (Immediately Invoked Function Expression)
print((lambda x, y: x + y)(10, 4))   # 14

# 3. Predicati Logici Booleani (Verifica Pari/Dispari con Modulo)
is_even = lambda x: x % 2 == 0
print(is_even(2))                    # True
print(is_even(3))                    # False

# 4. Suddivisione di Stringhe in Blocchi con List Comprehension (Chunking)
# Prende una sequenza x e una dimensione y, producendo porzioni a intervalli di y caratteri
blocks = lambda x, y: [x[i:i + y] for i in range(0, len(x), y)]
print(blocks("string", 2))           # ['st', 'ri', 'ng']

# 5. Trasformazione Caratteri in Valori ASCII (ord)
to_ord = lambda x: [ord(i) for i in x]
print(to_ord("ABCD"))                # [65, 66, 67, 68]

# Confronto con la Funzione Tradizionale Equivalente
def to_ord2(x):
    ret = []
    for i in x:
        ret.append(ord(i))
    return ret

print(to_ord2("ABCD"))               # [65, 66, 67, 68]
```
