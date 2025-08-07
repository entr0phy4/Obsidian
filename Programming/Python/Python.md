Python is a high-level, general-purpose programming language. It's design philosophy emphasizes code readability with the use of significant indentation.
[[Multi-paradigm]]

---

#Datatypes

- List

- Tuples

- string

- bitarray
	Also known as bitmaps, bit sets, bit strings, or bit vectors, provide an efficient way to store sequences of bits.

---

#Functions 
# zip()
Aggregates elements from multiple itarables ([[#Lists]], [[#Tuples]], etc)

```python
list1 = [1, 2, 3]
list2 = ['a', 'b', 'c', 'd']

zipped = zip(list1, list2)
print(list(zipped))
# Output: [(1, 'a'), (2, 'b'), (3, 'c')]
```
# encode(\<string\>)
This method returns an encoded version of the given [[#string]]
# ord(\<character\>)
Returns the `Unicode` code from a given character.

```python
print(ord('€'))  # Output: 8364
```

---

#Libraries

[[Scapy]]
[[bitarray]]

---
#Frameworks

[[FastApi]]
[[Flask]]
[[Django]]