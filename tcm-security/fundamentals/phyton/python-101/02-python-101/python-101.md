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
