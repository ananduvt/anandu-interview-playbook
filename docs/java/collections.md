# Collections Framework

## Collections Framework

**?? image**

* Iterable(I)
* Collection(I)
  * List(I)
    * ArrayList(C)
    * LinkedList(C)
    * Vector(C)
      * Stack(C)

  * Queue(I)
    * PriorityQueue(C)
    * Dque(I)
      * ArrayDque(C)

  * Set(I)
    * HashSet(C)
    * LinkedHashSet(C)

    * SortedSet(I)
      * TreeSet(C)

  * Map(I)
    * HashMap(C)
    * HashTable(C)
    * LinkedHashMap(C)
    *
    * SortedMap(I)
      * TreeMap(C)

**Collections** - utility Class ??

* addAll(\*)
* sort()
* shuffling()
* search - binary
* reverse
* rotate
* fill
* copy
* min and max

**Collections.sort - algorithm ?**

## ArrayList

[https://www.baeldung.com/java-list-capacity-array-size](https://www.baeldung.com/java-list-capacity-array-size)

When an ArrayList is created in Java, it is backed by an array. Initially, this array has a default capacity, often 10. When elements are added to the ArrayList, they are stored in this underlying array. If the ArrayList grows beyond its current capacity, a new, larger array is created, typically with a size that is 1.5 or 2 times the original size. The elements from the old array are then copied to the new array, and the old array is discarded, a process known as resizing. This ensures that ArrayList can accommodate a dynamic number of elements.

The memory occupied by an ArrayList includes the object header, the size of the internal array, and the memory used by the elements themselves. The header typically takes 12 bytes, and the array size depends on the number of elements and the data type. For example, an ArrayList of integers with 10 elements would require 40 bytes for the elements themselves (4 bytes per integer). When the ArrayList is resized, the memory usage increases to accommodate the new capacity.

ArrayList stores elements contiguously in memory, which allows for efficient access using indices. However, frequent resizing can lead to performance overhead due to the copying of elements. To mitigate this, it is often beneficial to initialize an ArrayList with an appropriate initial capacity if the number of elements is known in advance.

## Hashmap

[https://anmolsehgal.medium.com/java-hashmap-internal-implementation-21597e1efec3](https://anmolsehgal.medium.com/java-hashmap-internal-implementation-21597e1efec3)
[https://www.geeksforgeeks.org/internal-working-of-hashmap-java/](https://www.geeksforgeeks.org/internal-working-of-hashmap-java/)

In Java hashing, several key concepts govern the performance and behavior of hash-based data structures like HashMap. These concepts include capacity, threshold, rehashing, and collision.

**Capacity**:
The initial size of the internal array (bucket array) in a HashMap. The default capacity is typically 16, but it can be specified during HashMap instantiation.

**Threshold**:
A value that triggers the rehashing process. It's calculated as the product of the capacity and the load factor (default 0.75). When the number of entries in the HashMap exceeds the threshold, the HashMap resizes. For example, with a capacity of 16 and a load factor of 0.75, the threshold is 12.

**Rehashing**:
The process of resizing the internal array of a HashMap when the number of entries exceeds the threshold. It involves creating a new array with double the capacity, recalculating the hash codes for existing entries, and redistributing them into the new array. This ensures efficient retrieval times as the HashMap grows.

**Collision**:
Occurs when two or more keys produce the same hash code, resulting in them mapping to the same index in the internal array. HashMap handles collisions using techniques like separate chaining (using linked lists or trees) to store multiple entries at the same index.

### Load Factor

[https://www.baeldung.com/java-hashmap-load-factor#:\~:text=The%20load%20factor%20is%20the,code%20of%20already%20stored%20entries](https://www.baeldung.com/java-hashmap-load-factor#:~:text=The%20load%20factor%20is%20the,code%20of%20already%20stored%20entries).

In Java, the load factor is a crucial parameter associated with hash-based data structures like HashMap and HashSet. It determines when the data structure's internal capacity should be increased (rehashing). The load factor is a number between 0 and 1 (inclusive), with a default value of 0.75 in Java.

It represents the ratio of the number of elements stored in the hash table to the total number of buckets (capacity). When the load factor is reached, the capacity of the hash table is doubled, and all existing elements are rehashed into the new buckets. This process helps maintain the efficiency of the hash table's operations, such as insertion, deletion, and retrieval.

A lower load factor means that the hash table will be resized more frequently, leading to less collision and faster access times but also higher memory consumption. Conversely, a higher load factor means that the hash table will be resized less frequently, saving memory but potentially increasing collision and slowing down operations. The default value of 0.75 is generally considered a good balance between time and space efficiency.

**List**

* The List is an indexed sequence. - maintain order
* List allows duplicate elements
* Elements by their position can be accessed
* Multiple null elements can be stored.
* Implementations are ArrayList, LinkedList, Vector, Stack

**Set**

* The Set is an non-indexed sequence. - does not maintain order
* Set doesn’t allow duplicate elements.
* Position access to elements is not allowed.
* Null element can store only once.
* Implementations are HashSet, LinkedHashSet.

hash Map ??

List vs Set ??

Stack and queue

Hash Collision
Binary tree - use cases
Heap DS

clob , blob
Hashing
tokenization
