# Python 101
# 1. Variables and Data Types (Variabili e Tipi di Dato)

In Python le variabili vengono create nel momento stesso in cui viene assegnato loro un valore, senza dover dichiarare esplicitamente il tipo. Il tipo viene gestito dinamicamente dall'interprete.

---

### Codice Sorgente Completo (`variables-demo.py`)

```python
# Assegnazione di stringa e intero
name = "neut"
print(name)

name_length = 4
print(name_length)

# Assegnazione multipla sulla stessa riga
name, name_length = "neut", 4

# Verifica del tipo di dato
print(type(name))         # <class 'str'>
print(type(name_length))  # <class 'int'>

# Type casting (conversione di tipo esplicita)
name_length = int("4")
print(type(name_length))  # <class 'int'>

# Case sensitivity nei nomi delle variabili
name_length = 4
Name_length = 5
print(name_length)        # 4
print(Name_length)        # 5

# Liste (list) e unpacking
name_list = ["neut", "247CTF", "asd"]
print(type(name_list))    # <class 'list'>

name1, name2, name3 = name_list
print(name1)              # neut
print(name2)              # 247CTF
print(name3)              # asd

# Tuple (tuple) - sequenza ordinata e immutabile
name_tuple = ("neut", "247CTF")
print(type(name_tuple))   # <class 'tuple'>

# Dizionari (dict) - coppie chiave-valore
name_dictionary = {"neut": 4, "247CTF": 6}
print(type(name_dictionary)) # <class 'dict'>

# Booleani (bool)
name_boolean = True
print(type(name_boolean)) # <class 'bool'>

# Range (range) - sequenza di numeri
name_range = range(6)
print(type(name_range))   # <class 'range'>

# Bytes (bytes) - sequenza immutabile di byte
name_bytes = b"neut2"
print(type(name_bytes))   # <class 'bytes'>

# Stampa dei valori delle strutture dati
print(name_tuple)         # ('neut', '247CTF')
print(name_list)          # ['neut', '247CTF', 'asd']
print(name_dictionary)    # {'neut': 4, '247CTF': 6}
print(name_boolean)       # True
print(name_range)         # range(0, 6)
print(name_bytes)         # b'neut2'
```
# 2. Numbers (Numeri)

Python supporta diversi tipi di numeri, conversioni di base (esadecimale, ottale, binario) e funzioni matematiche built-in come valore assoluto e arrotondamento.

---

### Codice Sorgente Completo (`numbers-demo.py`)

```python
# Interi (int) e numeri a virgola mobile (float)
t1_int = 1
t1_float = 1.0

print(t1_int)                # 1
print(t1_float)              # 1.0

print(type(t1_int))          # <class 'int'>
print(type(t1_float))        # <class 'float'>

# Numeri complessi (complex) - la parte immaginaria usa il suffisso 'j'
t1_complex = 3.14j
print(t1_complex)            # 3.14j
print(type(t1_complex))      # <class 'complex'>

# Numeri esadecimali (prefisso 0x)
t1_hex = 0xa
print(t1_hex)                # 10
print(type(t1_hex))          # <class 'int'>

# Numeri ottali (prefisso 0o)
t1_octal = 0o10
print(t1_octal)              # 8
print(type(t1_octal))        # <class 'int'>

# Operazioni combinate tra basi diverse (decimale, esadecimale, ottale)
print(1 + 0x1 + 0o1)         # 3

# Valore assoluto con abs()
print(abs(4))                # 4
print(abs(-4))               # 4

# Arrotondamento con round()
print(round(8.4))            # 8
print(round(8.5))            # 8 (in Python 3 usa il "round half to even")
print(round(8.6))            # 9

# Conversioni di rappresentazione in binario ed esadecimale
print(bin(8))                # 0b1000
print(hex(8))                # 0x8

```
# 3. Strings and String Formatting (Stringhe e Formattazione)

In Python le stringhe sono sequenze immutabili di caratteri supportate da numerosi metodi integrati per la manipolazione, la ricerca, l'allineamento e la formattazione avanzata.

---

### Codice Sorgente Completo (`string-demo.py`)

```python
# 1. Definizione e Delimitatori
string1 = "I am a string!"
string2 = 'I am a string too!'

print(string1)
print(string2)

# Stringhe multi-linea con tripli apici
string3 = """I am a long
long
string!"""
print(string3)

# Gestione degli apici e caratteri di escape
string4 = "I'm a string"
print(string4)

string5 = 'I"m a string'
print(string5)

string6 = "I\"m a string\nI\"m on a newline!"
string6 = "\\ \x41\x42\x43"    # Escape del backslash e codici esadecimali ASCII (\x41=A, \x42=B, \x43=C)
print(string6)                 # Output: \ ABC

# Moltiplicazione e lunghezza
string7 = "aaaaaaaaaa"
print(string7)
string7 = "a" * 10
print(string7)
print(len(string7))            # 10

# 2. Metodi di Controllo, Ricerca e Trasformazione
print("neut" in string4)              # False (verifica presenza sottostringa)
print(string4.startswith("I"))       # True
print(string4.startswith("n"))       # False
print(string4.index("string"))       # 6 (indice di inizio della sottostringa)
print(string4.upper())               # Converte tutto in maiuscolo
print(string4.lower())               # Converte tutto in minuscolo

# Pulizia (strip), sostituzione (replace) e suddivisione (split)
messy_string = "Messy,string!"
print(messy_string)
print(messy_string.strip())
print(messy_string.replace("!", "?").strip())
print(messy_string.replace("string", "example"))

print(messy_string.split(","))       # ['Messy', 'string!']
print(messy_string.split())          # Divide per default sullo spazio: ['Messy,string!']

# Encoding in sequenza di byte
string4 = "I am a string!"
print(string4)
print(string4.encode())              # b'I am a string!'
print(string4.encode("utf-8"))       # b'I am a string!'

# Allineamento e padding (utile negli exploit script per payload/padding)
print(string4.rjust(25))             # Allinea a destra con spazi fino a lunghezza 25
print(string4.rjust(25, "X"))        # Riempie a sinistra con 'X'
print(string4.ljust(25))             # Allinea a sinistra con spazi
print(string4.ljust(25, "X"))        # Riempie a destra con 'X'

# 3. Concatenazione
print("I am " + "a string")
print("String 4 is " + str(len(string4)) + " characters long!")

print(1 + 1)                         # 2 (somma aritmetica)
print("1" + "1")                     # 11 (concatenazione di stringhe)
print(type("1" + "1"))               # <class 'str'>

# 4. Formattazione con .format()
print("string4 is {} characters long!".format(len(string4)))
print("{} {} {}".format(len(string4), 5.0, 0x12))
print("{0} {2} {1}".format(len(string4), 5.0, 0x12))               # Con indici posizionali
print("{length}".format(length=len(string4)))                      # Con argomenti con nome

# 5. Formattazione con f-strings (da Python 3.6+)
length = len(string4)
print(f"string4 is {length} characters long")
print(f"string4 is {length:.2f} characters long")                  # Formattazione decimale (2 decimali)
print(f"string4 is {length:.3f} characters long")                  # 3 decimali
print(f"string4 is {length:.4f} characters long")                  # 4 decimali

# Specificatori di base numerica nelle f-strings
print(f"string4 is {length:x} characters long")                    # Esadecimale ('e' per 14)
print(f"string4 is {length:b} characters long")                    # Binario ('1110' per 14)
print(f"string4 is {length:o} characters long")                    # Ottale ('16' per 14)

# 6. Formattazione in stile C (%-formatting)
print("string4 is %d characters long!" % len(string4))             # %d per intero
print("string4 is %f characters long!" % len(string4))             # %f per float
print("string4 is %x characters long!" % len(string4))             # %x per esadecimale
```
# 4. Booleans and Operators (Booleani e Operatori)

In Python i valori booleani sono rappresentati dalle costanti `True` e `False`. Python mette a disposizione operatori di confronto, logici, aritmetici, di assegnazione composta e bitwise (a livello di bit).

---

### Codice Sorgente Completo (`boolean-demo.py`)

```python
# 1. Definizione e Uguaglianza Booleana
valid = True
not_valid = False

print(valid)                         # True
print(not_valid)                     # False

print(valid == True)                 # True
print(not_valid == True)             # False

print(valid != True)                 # False
print(not_valid != True)             # True

# Operatore logico NOT
print(not valid)                     # False
print(not not_valid)                 # True

# 2. Operatori di Confronto
print((10 < 9) == True)              # False
print((10 == 10) == True)            # True
print((10 != 10) == True)            # False
print((10 >= 10) == True)            # True
print((10 <= 10) == True)            # True
print((10 > 9) == True)              # True

# Valutazione diretta delle espressioni di confronto
print((10 < 9))                      # False
print((10 == 10))                    # True
print((10 != 10))                    # False
print((10 >= 10))                    # True
print((10 <= 10))                    # True
print((10 > 9))                      # True

print("-----")

# 3. Operatori Logici (AND, OR) e Valutazione Booleana
print(10 > 5 and 10 < 5)             # False (entrambe devono essere True)
print(10 > 5 or 10 < 5)              # True (almeno una deve essere True)

# Casting a bool di interi
print(bool(0))                       # False (lo 0 numerico è falsy)
print(bool(1))                       # True  (i numeri diversi da 0 sono truthy)

print(bool(0) == False)              # True
print(bool(1) == True)               # True

# 4. Operatori Aritmetici
print(10 + 10)                       # 20 (Addizione)
print(10 - 10)                       # 0 (Sottrazione)
print(10 / 10)                       # 1.0 (Divisione standard - produce sempre un float)
print(10 // 10)                      # 1 (Divisione intera - floor division)

print(10 / 3)                        # 3.3333333333333335
print(10 // 3)                       # 3
print(10 % 3)                        # 1 (Modulo - resto della divisione)

print(10 * 10)                       # 100 (Moltiplicazione)
print(10 ** 10)                      # 10000000000 (Esponenziazione / Potenza)
print(10 % 10)                       # 0

# 5. Operatori di Assegnazione Composta
x = 10
print(x)                             # 10
x = x + 1
print(x)                             # 11
x += 1                               # Equivalente a x = x + 1
print(x)                             # 12
x -= 1                               # Equivalente a x = x - 1
print(x)                             # 11
x *= 5                               # Equivalente a x = x * 5
print(x)                             # 55
x /= 5                               # Equivalente a x = x / 5
print(x)                             # 11.0

# 6. Operazioni Bitwise (Shift a livello di bit)
x = 13
print(bin(x))                        # 0b1101
# Rimuove il prefisso '0b' con slicing [2:] e allinea a 4 bit con '0'
print(bin(x)[2:].rjust(4, "0"))       # 1101

# Shift a destra di 1 bit (x >> 1): 1101 diventa 0110 (valore decimale: 6)
print(bin(x >> 1)[2:].rjust(4, "0")) # 0110
```
# 5. Tuples (Tuple)

Le tuple sono sequenze ordinate e immutabili racchiuse tra parentesi tonde `()`. Possono contenere elementi duplicati e tipi di dato misti, supportando indicizzazione, slicing, spacchettamento (unpacking) e concatenazione[cite: 15, 16].

---

### Codice Sorgente Completo (`tuple-demo.py`)

```python
# 1. Creazione e Verifica del Tipo
tuple_items = ("item1", "item2", "item3")
print(tuple_items)
print(type(tuple_items))             # <class 'tuple'>

tuple_numbers = (1, 2, 3)
print(tuple_numbers)
print(type(tuple_numbers))           # <class 'tuple'>

# Tupla con singolo elemento (richiede la virgola finale) e moltiplicazione
tuple_repeat = ('Combine!',) * 4
print(tuple_repeat)                  # ('Combine!', 'Combine!', 'Combine!', 'Combine!')
print(type(tuple_repeat))            # <class 'tuple'>

# Tipi eterogenei e annidati (tuple all'interno di tuple)
mixed_tuple = ("A", 1, ("A", 1))
print(mixed_tuple)
print(type(mixed_tuple))             # <class 'tuple'>

# Concatenazione tra tuple con '+'
tuple_combined = tuple_items + tuple_numbers
print(tuple_combined)                # ('item1', 'item2', 'item3', 1, 2, 3)
print(type(tuple_combined))          # <class 'tuple'>

# 2. Unpacking (Spacchettamento)
item1, item2, item3 = tuple_items
print(item1)                         # item1
print(item2)                         # item2
print(item3)                         # item3

# 3. Controllo Appartenenza e Metodi di Ricerca
print("item2" in tuple_items)        # True
print("item3" in tuple_items)        # True
print("item4" in tuple_items)        # False

# Ricerca dell'indice di un valore
print(tuple_items.index("item2"))    # 1

# 4. Indicizzazione e Slicing
tuple_items = ("item1", "item2", "item3")
print(tuple_items[0])                # item1 (primo elemento)
print(tuple_items[1])                # item2
print(tuple_items[2])                # item3

print(len(tuple_items))              # 3 (lunghezza della tupla)

# Indici negativi (partendo dal fondo)
print(tuple_items[-1])               # item3 (ultimo elemento)
print(tuple_items[-2])               # item2 (penultimo elemento)

# Slicing [inizio:fine] (esclude l'indice di fine)
print(tuple_items[0:2])              # ('item1', 'item2')
print(tuple_items[:2])               # ('item1', 'item2')
print(tuple_items[-3:-1])            # ('item1', 'item2')

# Slicing applicato alle stringhe (concetto identico alle tuple)
string1 = "I am a string!"
print(string1[0:4])                  # I am
print(string1[-1])                   # !
```
# 6. Lists (Liste)

Le liste in Python sono collezioni ordinate, mutabili e flessibili delimitate da parentesi quadre `[]`. A differenza delle tuple, le liste permettono di modificare, aggiungere, eliminare e ordinare i propri elementi sul posto[cite: 17, 18, 19].

---

### Codice Sorgente Completo (`list-demo.py`)

```python
# 1. Creazione e Tipi di Dato Misti
list1 = ["A", "B", "C", "D", "E", "F"]
print(list1)

# Lista contenente tipi misti: str, int, float, liste annidate, tuple, bool
list2 = ["A", 1, 2.0, ["A"], [], list(), ("A"), False]
print(list2)
print(type(list2))                  # <class 'list'>

# 2. Indicizzazione e Annidamento
print(list1[0])                     # 'A' (primo elemento)
print(list1[-1])                    # 'F' (ultimo elemento)
print(list2[3][0])                  # 'A' (primo elemento della sottolista in indice 3)
print(list2[3][-1])                 # 'A'

# 3. Modifica, Inserimento e Rimozione
list1[0] = "X"                      # Sovrascrittura elemento per indice
print(list1)                        # ['X', 'B', 'C', 'D', 'E', 'F']

del list1[0]                        # Eliminazione per indice con l'istruzione del
print(list1)                        # ['B', 'C', 'D', 'E', 'F']

list1.insert(0, "A")                # Inserimento di 'A' in posizione 0
print(list1)                        # ['A', 'B', 'C', 'D', 'E', 'F']

del list1[0]
print(list1)                        # ['B', 'C', 'D', 'E', 'F']

# Prepend tramite concatenazione
list1 = ["A"] + list1
print(list1)                        # ['A', 'B', 'C', 'D', 'E', 'F']

# 4. Aggiunta in Coda ed Estrazione
list1.append("G")                   # Aggiunge un elemento in coda
print(list1)

print(max(list1))                   # 'G' (valore massimo lessicografico)
print(min(list1))                   # 'A' (valore minimo lessicografico)

# Ricerca indice e dereferenziazione
print(list1.index("C"))             # 2
print(list1[list1.index("C")])      # 'C'

# Inversione degli elementi
list1.reverse()                     # Inverte la lista sul posto (in-place)
print(list1)

list1 = list1[::-1]                 # Inversione tramite slicing con passo negativo
print(list1)

# Conteggio e rimozione con pop()
print(list1.count("A"))             # 1 (conta occorrenze)
list1.append("A")
print(list1)
print(list1.count("A"))             # 2

list1.pop()                         # Rimuove e restituisce l'ultimo elemento
print(list1)

# 5. Estensione e Svuotamento
list3 = ["H", "I", "J"]
print(list3)

list1.extend(list3)                 # Estende la lista concatenando list3 in coda
print(list1)

list1.clear()                       # Rimuove tutti gli elementi
print(list1)                        # []

# 6. Ordinamento (sort)
list4 = [8, 12, 5, 6, 17, 2]
print(list4)

list4.sort()                        # Ordinamento crescente sul posto
print(list4)                        # [2, 5, 6, 8, 12, 17]

list4.sort(reverse=True)            # Ordinamento decrescente
print(list4)                        # [17, 12, 8, 6, 5, 2]

# 7. Riferimento vs Copia Superficiale (Shallow Copy)
# Assegnazione per riferimento (stesso oggetto in memoria)
list5 = list4
print(list4)
print(list5)

list5[2] = "X"                      # Modifica list5...
print(list5)
print(list4)                        # ...e modifica anche list4!

# Copia indipendente con .copy()
list6 = list4.copy()
print(list4)
print(list6)

list6[2] = "A"                      # Modifica solo la copia list6
print(list6)                        # [17, 12, 'A', 6, 5, 2]
print(list4)                        # list4 rimane inalterata: [17, 12, 'X', 6, 5, 2]

# 8. Mappatura e Trasformazione di Tipo (map)
list7 = ["1", "2", "3"]
print(list7)

# Converte ogni stringa della lista in float
list8 = list(map(float, list7))
print(list8)                        # [1.0, 2.0, 3.0]
```
