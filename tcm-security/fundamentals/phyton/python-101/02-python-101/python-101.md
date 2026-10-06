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
