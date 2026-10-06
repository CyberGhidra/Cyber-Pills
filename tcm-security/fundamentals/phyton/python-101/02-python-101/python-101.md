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
