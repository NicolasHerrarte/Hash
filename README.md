# Generic Chained Hash Table & Dictionary

## Highlights

* **Hybrid Chained Hash Table**: Combines a traditional bucket-based table for $O(1)$ lookups with contiguous dynamic array storage to improve cache locality.
* **Arena-Style Object Storage**: Decouples hash nodes from data by storing keys and values in dedicated `dynarray` structures for safer memory management.
* **Open-Addressing Hole Reuse**: Features an efficient deletion strategy using a `holes` stack to track and reuse indices from deleted entries, preventing fragmentation.
* **Dual-Mode Generic Interface**: Supports both simple hash sets (single storage) and dictionary mappings (separate key/value storage) via configurable strides.
* **Advanced Hash Functions**: Includes specialized implementations of the **DJB33** algorithm for strings and a 64-bit mixing function for integer keys.
* **Collision Resolution via Chaining**: Employs a robust linked-list approach within buckets to ensure correctness even under high-load factors.
* **Strict Memory Control**: Built with manual memory management in C, leveraging a pointer-based interface for custom equality and hashing logic.

---

## Overview

This implementation provides a flexible, generic Hash and Dictionary utility based on the principles of efficient memory layout and manual resource control. It is designed to support complex compiler tasks, such as building Symbol Tables or managing LR(1) Canonical Collections, where memory ownership must be explicit.

---

## Usage

### 1. Creating a Dictionary (String Mapping)
To create a dictionary that maps strings to values, use the `dynadict` macros. This configuration uses the DJB33 hash algorithm and string equality checks automatically.

```c
Hash symbol_table = dynadict_create(1024, int);

char* key = "variable_name";
int value = 100;

dynadict_add(symbol_table, key, value);

int* found = (int*)dynadict_get(symbol_table, key);
if (found) {
    printf("Value: %d\n", *found);
}

```
### 2. Creating a Generic Hash Set
For storing unique objects, use the hash_create macro. You must provide 2 hashing functions compatible with your data type. An equality function and a hashing function. Both hashing function also need `void*` as parameters.

```c
Hash int_set = hash_create(512, int, _hash_int);

int my_val = 1337;

hash_add(int_set, my_val, NULL);

if (hash_in(int_set, my_val, NULL)) {
    printf("Value is in the set!\n");
}

```

## Important Recommendations and Warnings
* **Memory Ownership and Cleanup**: The Hash structure manages memory for its internal table, obj_storage, key_storage, and holes. While internal Node structs are manually allocated during insertion and must be freed during destruction, the table only copies pointers for complex types like char*. You must free external referenced memory manually.

* **Pointer Stability**: Because objects are stored in a dynarray, the physical memory address of an item may change if the storage array resizes during a push operation. Always use _hash_get to retrieve the current valid address of an item rather than caching a pointer across multiple insertions.

* **Collision Performance**: To maintain $O(1)$ performance, initialize the node_amount (capacity) to be approximately 1.5x to 2x your expected number of elements. A small table size relative to the count will degrade performance to $O(n)$ as linked lists grow.