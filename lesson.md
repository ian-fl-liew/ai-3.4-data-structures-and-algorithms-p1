# Lesson 3.4: Data Structures and Algorithms (Part 1)

## Lesson Overview
This lesson introduces the data structures you will use every day in Java — Arrays, ArrayList, LinkedList, ArrayDeque, HashMap, and HashSet, along with their ordered and sorted variants. Students will learn how these structures organise and store data, how to choose the right one for a given problem, and how modern Java (Java 17 and 21) has simplified working with them. By the end of this lesson, learners will be able to select and use the appropriate data structure with confidence.

**Module:** 3.4
**Duration:** 3 hours
**Prerequisites:** Basic Java syntax, variables, operators, loops, control flow, classes and objects.

---

## Lesson Objectives

By the end of this lesson, students will be able to:
- Describe the Java Collections Framework and the difference between an interface and an implementation.
- Differentiate between linear and hash-based data structures.
- Implement and manipulate Arrays, ArrayLists, LinkedLists, ArrayDeques, HashMaps, and HashSets.
- Use records to store data safely in Sets and as Map keys.
- Apply the appropriate data structure for a given problem.

---

## Part 1: Introduction to Data Structures

Data structures are containers that organise and store data so it can be accessed and modified efficiently. Choosing the right data structure affects how quickly your program can process data. Retrieving a record from a list of 10 items is trivial, but the same operation across millions of records is where the choice starts to matter.

Data structures fall into **three categories**:

| Category | Structures | How data is stored |
|----------|-----------|-------------------|
| **Linear** | Array, ArrayList, LinkedList, ArrayDeque | Sequentially, one after another |
| **Hash-Based** | HashMap, HashSet and their variants | By hash code, no sequence |
| **Non-Linear** | Tree, Binary Tree, Graph | Hierarchically or as a network |

Hash-based structures are their own category. They are neither linear nor non-linear, because elements are placed according to their hash code rather than by position or by hierarchy.

**This lesson covers linear and hash-based structures. Non-linear structures are covered in Lesson 3.5.**

---

### A Quick Word on Performance

Throughout this lesson you will see notations like **O(1)** and **O(n)**. These are shorthand for how an operation behaves as your data grows:

- **O(1)** — the same speed regardless of how much data you have. Like flipping to a bookmarked page. The size of the book does not matter.
- **O(n)** — the time grows in proportion to the data. Double the data, double the time. Like checking every locker in a hallway one by one.
- **O(log n)** — grows, but very slowly. Like finding a word in a dictionary, where each guess eliminates half the remaining pages.

That is all you need for today. This is vocabulary, not mathematics — we use it to compare structures quickly. Lesson 3.5 goes into this in more depth.

---

## Part 2: The Java Collections Framework

Java already provides a ready-made toolbox of data structures. You do not need to build a resizable list or a hash table yourself — Java has written, tested, and optimised them for you. This toolbox is called the **Java Collections Framework**, often shortened to JCF.

Everything in this lesson except plain arrays comes from this framework.

### Interfaces and Implementations

The Collections Framework is organised into two layers.

**Interfaces** describe *what a structure does*. `List`, `Set`, `Queue`, and `Map` are interfaces. Think of an interface as a category of behaviour:

- `List` — holds elements in order, duplicates allowed
- `Set` — holds unique elements only
- `Queue` — holds elements for processing in a particular order
- `Map` — holds key and value pairs

**Classes** are the actual implementations you create objects from. `ArrayList`, `LinkedList`, `HashSet`, `TreeMap`, `ArrayDeque` are all classes. These are what you import and instantiate.

> **Note:** Interfaces are covered fully in a later lesson. For now, the practical rule is enough: an interface is the category, a class is the thing you actually create with `new`.

### The Structure

```
Iterable
   └── Collection
         ├── List   →  ArrayList, LinkedList
         ├── Set    →  HashSet, LinkedHashSet, TreeSet
         └── Queue  →  ArrayDeque, LinkedList

Map (separate from Collection)
   └── HashMap, LinkedHashMap, TreeMap
```

Notice that `Map` sits outside the `Collection` tree. This is deliberate — a `Collection` holds single elements, while a `Map` holds pairs of elements. They are related in purpose but not in structure.

### Declaring by Interface

In professional Java code, you declare the variable using the **interface** and create the object using the **class**:

```java
List<String> productNames = new ArrayList<>();
Map<String, Integer> stockLevels = new HashMap<>();
Set<String> categories = new HashSet<>();
```

The advantage is flexibility. If you later decide `LinkedList` suits your access pattern better, you change one word at the point of creation. Every other line of code that uses that variable continues to work unchanged.

You will see this pattern in every production Java codebase.

### Modern Shorthand: `var`

Since Java 10, you can let Java infer the type of a local variable:

```java
var productNames = new ArrayList<String>();
```

This is common in modern codebases. It is only for local variables inside methods, and the type must still be clear from the right hand side. We will use it occasionally so you recognise it when you see it.

---

## Part 3: Linear Data Structures

### Arrays

An **Array** is a fixed-size, indexed collection of elements of the same data type. It stores data in contiguous memory, which allows direct access by index.

```java
int[] productPrices = {999, 499, 299, 199, 129};

for (int i = 0; i < productPrices.length; i++) {
    System.out.println("Index " + i + ": " + productPrices[i]);
}
```

Arrays give **O(1)** access because the position of any element can be calculated directly. The trade-off is rigidity — the size is fixed at creation and cannot grow. Searching for a value without knowing its index requires checking each element, which is **O(n)**.

**Two things worth knowing:**

Arrays use `.length` — a property, with no brackets. Collections use `.size()` — a method, with brackets. This catches people out regularly.

An array is an object. When you write `new int[5]`, an array object is created on the **heap**, and the variable holding it stores a reference to that object. This is the same model used for every object in Java.

---

### ArrayList

An **ArrayList** is a resizable list. Internally it is still backed by an array, but Java manages the growing for you.

When the internal array fills up, Java creates a new larger array (roughly 1.5 times the size) and copies the elements across. This copy is **O(n)**, but it happens rarely. Averaged across many additions, `add()` is described as **O(1) amortised**.

If you know roughly how many elements you will add, you can avoid repeated resizing:

```java
List<String> productNames = new ArrayList<>(10000);
```

#### Core Operations

```java
List<String> productNames = new ArrayList<>();
productNames.add("Laptop");
productNames.add("Phone");
productNames.add("Tablet");

System.out.println(productNames.get(0));        // Laptop
System.out.println(productNames.size());        // 3
System.out.println(productNames.contains("Phone")); // true

productNames.remove("Phone");
System.out.println(productNames);               // [Laptop, Tablet]
```

#### Beyond the Basics

These are the methods you will actually reach for in real code.

```java
List<String> productNames = new ArrayList<>();
productNames.add("Laptop");
productNames.add("Phone");
productNames.add("Tablet");
productNames.add("Monitor");

// set — replace the element at a position
productNames.set(1, "Smartphone");
System.out.println(productNames);          // [Laptop, Smartphone, Tablet, Monitor]

// add at a specific index — shifts everything after it to the right
productNames.add(2, "Keyboard");
System.out.println(productNames);          // [Laptop, Smartphone, Keyboard, Tablet, Monitor]

// indexOf — find the position of a value, returns -1 if not present
System.out.println(productNames.indexOf("Tablet"));   // 3
System.out.println(productNames.indexOf("Printer"));  // -1

// subList — a view of part of the list
System.out.println(productNames.subList(0, 3));  // [Laptop, Smartphone, Keyboard]

// addAll — append another collection
productNames.addAll(List.of("Mouse", "Webcam"));

// removeIf — remove everything matching a condition
productNames.removeIf(p -> p.startsWith("M"));
System.out.println(productNames);

// sort the list alphabetically
Collections.sort(productNames);
System.out.println(productNames);

// reverse order
Collections.sort(productNames, Collections.reverseOrder());
```

> **Note on `removeIf()`:** the part inside the brackets, `p -> p.startsWith("M")`, is a **lambda**. Read it as "for each element p, is it true that p starts with M". Lambdas are covered properly in a later lesson. For now, recognise the shape.

#### Creating Lists

```java
// Empty and mutable
List<String> emptyList = new ArrayList<>();

// Fixed and unchangeable — Java 9 onwards
List<String> fixedList = List.of("Laptop", "Phone");

// Mutable copy of a fixed list
List<String> mutableCopy = new ArrayList<>(List.of("Laptop", "Phone"));
```

`List.of()` creates an **immutable** list. Calling `.add()` on it throws an `UnsupportedOperationException` at runtime. This is intentional — immutable collections are safer to pass around because nothing can modify them unexpectedly. When you need to modify, wrap it in `new ArrayList<>()` as shown above.

---

### 👨‍💻 Activity 1: Managing a Product List **(10 minutes)**

Create an `ArrayList` of product names and use it to do the following:

1. Add at least six products to the list.
2. Replace the product at index 2 with a different name using `set()`.
3. Insert a new product at index 0 so it becomes the first item.
4. Print the position of one specific product using `indexOf()`.
5. Print only the first three products using `subList()`.
6. Remove all products whose name is longer than 7 characters using `removeIf()`.
7. Sort the remaining list alphabetically and print it.

---

### LinkedList

A **LinkedList** stores elements as **nodes**. Each node holds the data and a reference to the next node. The nodes do not need to sit next to each other in memory — Java follows the chain of references.

```java
LinkedList<String> recentlyViewed = new LinkedList<>();

recentlyViewed.add("Mouse");
recentlyViewed.add("Keyboard");
recentlyViewed.add("Monitor");

recentlyViewed.addFirst("Charger");
recentlyViewed.removeLast();

System.out.println(recentlyViewed);   // [Charger, Mouse, Keyboard]
```

`addFirst()` and `removeLast()` are **O(1)** — adding or removing at either end only requires updating a couple of references, with no shifting.

The trade-off is access. Getting the element at position 5 means starting at the head and walking node by node, which is **O(n)**.

> **Note:** LinkedList checks whether your index is nearer the head or the tail and traverses from the closer side. So `get(0)` and the last element are effectively instant. It is the middle that costs you.

---

### ArrayDeque

A **Deque** (pronounced "deck") is a double-ended queue — you can add and remove from both ends. `ArrayDeque` is the standard implementation.

```java
Deque<String> recentlyViewed = new ArrayDeque<>();

recentlyViewed.addLast("Mouse");
recentlyViewed.addLast("Keyboard");
recentlyViewed.addLast("Monitor");

System.out.println(recentlyViewed.peekFirst());   // Mouse — look without removing
System.out.println(recentlyViewed.pollFirst());   // Mouse — remove and return
System.out.println(recentlyViewed);               // [Keyboard, Monitor]

recentlyViewed.addFirst("Charger");
System.out.println(recentlyViewed);               // [Charger, Keyboard, Monitor]
```

**Common methods:**

| Method | What it does |
|--------|-------------|
| `addFirst()` / `addLast()` | Add to the front or the back |
| `peekFirst()` / `peekLast()` | Look at an element without removing it |
| `pollFirst()` / `pollLast()` | Remove and return an element |
| `push()` / `pop()` | Stack behaviour — add and remove from the front |

#### Which One Should You Use?

This matters in real code:

- **`Stack`** is a legacy class from Java 1.0. It is synchronised, which makes it slower, and it is no longer recommended. Do not use it in new code.
- **`LinkedList`** works as a queue, but because its nodes are scattered in memory, the processor cannot cache them efficiently. In practice it is usually slower than the alternative.
- **`ArrayDeque`** is the current recommendation for stack, queue, and deque behaviour. It is backed by an array, which the processor handles far better.

The practical rule: **if you need a stack or a queue, reach for `ArrayDeque`.**

---

## Part 4: Hash-Based Data Structures

### HashMap, LinkedHashMap and TreeMap

A **HashMap** stores data as **key and value** pairs. Each key is unique. The key is passed through a hash function which produces a number, and that number determines where the value is stored internally. This is what makes lookups fast regardless of how much data you have.

All three map types implement the `Map` interface. They store key and value pairs and require unique keys. They differ in ordering and speed.

| | HashMap | LinkedHashMap | TreeMap |
|--|---------|---------------|---------|
| Order | No guaranteed order | Insertion order | Sorted by key |
| Speed | O(1) | O(1) | O(log n) |
| Use when | Speed matters | Order matters | Sorted keys needed |

---

#### HashMap

```java
Map<String, Integer> studentScores = new HashMap<>();
studentScores.put("Alice", 85);
studentScores.put("Bob", 92);
studentScores.put("Charlie", 78);

// getOrDefault — returns a fallback if the key is missing
System.out.println(studentScores.getOrDefault("David", 0));   // 0

// putIfAbsent — only adds if the key is not already there
studentScores.putIfAbsent("Alice", 99);
System.out.println(studentScores.get("Alice"));               // 85, unchanged

System.out.println(studentScores);   // order not guaranteed
```

> **Key rule:** keys must be unique. Putting the same key again **overwrites** the existing value rather than creating a second entry. Values may repeat freely.

#### Iterating a Map

```java
for (Map.Entry<String, Integer> entry : studentScores.entrySet()) {
    System.out.println(entry.getKey() + " scored " + entry.getValue());
}
```

`entrySet()` gives you each pair as a `Map.Entry`, so you get both the key and the value in one pass. You can also use `keySet()` for keys only or `values()` for values only.

#### Adding to a List Inside a Map — the Manual Way

Before looking at `computeIfAbsent()`, it helps to see what it replaces.

When a Map holds Lists as its values, you cannot add an item until a List actually exists for that key. So it takes two steps: create the List, then add to it.

```java
Map<String, List<String>> byCategory = new HashMap<>();

// Step 1 — create the empty list once, and store it under the key
byCategory.put("Electronics", new ArrayList<>());

// Step 2 — get that list back, and add as many items as you like
byCategory.get("Electronics").add("Laptop");
byCategory.get("Electronics").add("Phone");
byCategory.get("Electronics").add("Tablet");

System.out.println(byCategory);
// {Electronics=[Laptop, Phone, Tablet]}
```

`put()` creates the key and stores an empty list as its value. `get()` hands that list back so you can add to it.

#### `computeIfAbsent()`

This one is worth learning properly because it appears constantly in real code.

Suppose you want to group products by category. Without `computeIfAbsent()` you have to check whether the list exists first:

```java
Map<String, List<String>> byCategory = new HashMap<>();

// The long way
if (!byCategory.containsKey("Electronics")) {
    byCategory.put("Electronics", new ArrayList<>());
}
byCategory.get("Electronics").add("Laptop");
```

With `computeIfAbsent()`, that becomes one line:

```java
Map<String, List<String>> byCategory = new HashMap<>();

byCategory.computeIfAbsent("Electronics", k -> new ArrayList<>()).add("Laptop");
byCategory.computeIfAbsent("Electronics", k -> new ArrayList<>()).add("Phone");
byCategory.computeIfAbsent("Furniture", k -> new ArrayList<>()).add("Desk");

System.out.println(byCategory);
// {Electronics=[Laptop, Phone], Furniture=[Desk]}
```

Read it as: "get the list for this key, and if there isn't one yet, create an empty list first — then add to it."

This pattern — a Map whose values are Lists — is extremely common. Any time you need to group things, this is the idiom.

---

#### LinkedHashMap

```java
Map<String, Integer> examResults = new LinkedHashMap<>();
examResults.put("Alice", 85);
examResults.put("Bob", 92);
examResults.put("Charlie", 78);

for (Map.Entry<String, Integer> entry : examResults.entrySet()) {
    System.out.println(entry.getKey() + " scored " + entry.getValue());
}
// Always in insertion order:
// Alice scored 85
// Bob scored 92
// Charlie scored 78
```

---

#### TreeMap

```java
TreeMap<String, Integer> rankings = new TreeMap<>();
rankings.put("Charlie", 78);
rankings.put("Alice", 85);
rankings.put("Bob", 92);

System.out.println(rankings.firstKey());          // Alice
System.out.println(rankings.lastKey());           // Charlie
System.out.println(rankings.headMap("Charlie"));  // {Alice=85, Bob=92}

System.out.println(rankings);   // {Alice=85, Bob=92, Charlie=78} always sorted
```

`firstKey()`, `lastKey()`, and `headMap()` exist only on TreeMap, because only TreeMap guarantees an order to work with.

By default TreeMap sorts by **natural order** — alphabetical for Strings, numerical for numbers.

---

### 👨‍💻 Activity 2: Grouping Products by Category **(10 minutes)**

Create a `Map<String, List<String>>` to group product names under their category.

1. Using `computeIfAbsent()`, add at least six products across three categories.
2. Print the whole map.
3. Print just the products in one category using `get()`.
4. Use `getOrDefault()` to look up a category that does not exist, and show that it returns an empty list rather than `null`.
5. Change your `HashMap` to a `LinkedHashMap` and run it again. Describe what changed in the output.

---

### HashSet, LinkedHashSet and TreeSet

A **HashSet** stores unique elements. Duplicates are silently ignored. Internally a HashSet is built on top of a HashMap — each element you add becomes a key in that map, and the value is an unused placeholder. That is the whole mechanism.

| | HashSet | LinkedHashSet | TreeSet |
|--|---------|---------------|---------|
| Order | No guaranteed order | Insertion order | Sorted |
| Speed | O(1) | O(1) | O(log n) |
| Use when | Speed and uniqueness | Order and uniqueness | Sorted uniqueness |

---

#### HashSet

```java
Set<String> categories = new HashSet<>();
categories.add("Electronics");
categories.add("Clothing");
categories.add("Electronics");   // duplicate, silently ignored

System.out.println(categories.contains("Clothing"));   // true
System.out.println(categories.size());                 // 2

categories.removeIf(c -> c.startsWith("E"));
System.out.println(categories);                        // [Clothing]
```

Notice that adding a duplicate does not throw an error. Uniqueness is enforced automatically — you never need to check `contains()` before adding.

#### Set Operations

```java
Set<String> allCategories = new LinkedHashSet<>(
    List.of("Electronics", "Clothing", "Furniture", "Sports"));

Set<String> activeCategories = Set.of("Clothing", "Sports");

// retainAll — keep only what appears in both
Set<String> active = new LinkedHashSet<>(allCategories);
active.retainAll(activeCategories);
System.out.println(active);        // [Clothing, Sports]

// removeAll — remove everything that appears in the other set
Set<String> inactive = new LinkedHashSet<>(allCategories);
inactive.removeAll(activeCategories);
System.out.println(inactive);      // [Electronics, Furniture]

// addAll — merge, duplicates ignored
Set<String> combined = new LinkedHashSet<>(allCategories);
combined.addAll(Set.of("Books", "Clothing"));
System.out.println(combined);
```

`retainAll()`, `removeAll()`, and `addAll()` give you set intersection, difference, and union without writing any loops.

---

#### TreeSet

```java
TreeSet<String> sortedCategories = new TreeSet<>(
    List.of("Furniture", "Electronics", "Clothing"));

System.out.println(sortedCategories.first());                    // Clothing
System.out.println(sortedCategories.last());                     // Furniture
System.out.println(sortedCategories.headSet("Furniture"));       // [Clothing, Electronics]
System.out.println(sortedCategories.subSet("Clothing", "Furniture")); // [Clothing, Electronics]

System.out.println(sortedCategories);   // [Clothing, Electronics, Furniture] always sorted
```

---

## Part 5: Working With Your Own Objects

Everything so far has stored Strings and numbers. Real applications store your own objects — products, customers, orders. That introduces something you need to know about.

### The Problem

Suppose you write an ordinary class to hold an item — an id, a name, and a price. Nothing unusual about it.

Now put two identical items into a Set:

```java
Set<Item> items = new HashSet<>();
items.add(new Item("P1", "Laptop", 999.0));
items.add(new Item("P1", "Laptop", 999.0));   // exactly the same data

System.out.println(items.size());   // 2 — the duplicate was NOT removed
```

Same id, same name, same price. Yet the Set kept both.

**Why?** Because by default Java compares objects by their **memory address**, not by the values inside them. You called `new` twice, so there are two objects sitting at two different addresses. As far as Java is concerned they are two different things, and the Set's uniqueness check never had a chance.

### What Is a Record?

A **record is just a class** — a shorter kind of class, built for one job: holding data.

Think about what you normally write for a data class. Private fields, a constructor, a getter for every field, a `toString()`. Thirty or forty lines, almost all of it boilerplate.

A record collapses all of that into one line:

```java
record Item(String id, String name, double price) { }
```

Java reads that and generates the fields, the constructor, the getters, `toString()`, and the logic for comparing one item to another.

When you write `new Item("P1", "Laptop", 999.0)` you are creating an ordinary object, exactly as with any class. A thousand of those is a thousand objects.

> **Note:** Records are covered fully in a later lesson. For now you only need to know that a record is a short way to write a data class, and that it comes with correct comparison behaviour built in.

### The Fix

Run exactly the same code, but with `Item` declared as a record:

```java
record Item(String id, String name, double price) { }

Set<Item> items = new HashSet<>();
items.add(new Item("P1", "Laptop", 999.0));
items.add(new Item("P1", "Laptop", 999.0));

System.out.println(items.size());   // 1 — duplicate correctly removed
```

The duplicate disappears.

**The difference in one sentence:** a record knows how to compare itself by its values. A plain class does not, so the Set saw two different things.

### Two Things to Know

**Getters have no `get` prefix.** It is `item.name()`, not `item.getName()`.

```java
Item laptop = new Item("P1", "Laptop", 999.0);

System.out.println(laptop.name());    // Laptop
System.out.println(laptop.price());   // 999.0
System.out.println(laptop);           // Item[id=P1, name=Laptop, price=999.0]
```

**Records are immutable.** Once created, the values cannot change. There are no setters. If you need different values, you create a new object. That is exactly why records are safe as Map keys — a key whose value changed underneath you would no longer be found where it was stored.

### When Does This Actually Matter?

The rule is narrower than "always use records". It applies to objects Java has to **compare**:

- **Set elements** — the Set compares elements to enforce uniqueness
- **Map keys** — the Map compares keys to find the right entry

It does not matter for **Map values** or **List elements**, because those are never compared for uniqueness.

> **The rule:** if Java needs to compare your object to decide "have I seen this one before?", make it a record.

> **Note:** If you created a file earlier in this lesson that declares a class with the same name, delete or rename it first. Two files declaring the same type in one folder will not compile.

### 👨‍💻 Activity 3: The Product Catalogue **(10 minutes)**

1. Create a `record CatalogueItem(String id, String name, double price)`.
2. Create a `List<CatalogueItem>` and add at least six items. Include **two entries that are exactly identical**.
3. Print the list and its size, and confirm the duplicate is present.
4. Create a `Set<CatalogueItem>` from your list. Print its size and confirm the duplicate has been removed automatically.

> **Note:** the record is named `CatalogueItem` rather than `Item` or `Product` so that it does not clash with any file you created earlier in this lesson.

---

## Part 6: Sorting With Comparators (Optional)

Java gives you two methods for sorting, and which one you use depends on what you are sorting.

- **`Arrays.sort()`** — for arrays. `int[]`, `String[]`, anything with square brackets.
- **`Collections.sort()`** — for Lists. ArrayList, LinkedList.

Both sort **in place**. They modify what you give them and return nothing. Writing
`numbers = Arrays.sort(numbers);` is a compile error, and it is a common first attempt.

```java
int[] prices = {999, 299, 499};
Arrays.sort(prices);                  // [299, 499, 999]

List<String> names = new ArrayList<>(List.of("Phone", "Laptop", "Tablet"));
Collections.sort(names);              // [Laptop, Phone, Tablet]
```

Both of those worked without any extra effort because Strings and numbers already know how to
order themselves. That built-in ordering is called **natural ordering** — alphabetical for
Strings, numerical for numbers.

Your own objects do not have one. A `CatalogueItem` has an id, a name and a price, and Java has no
way of knowing which one you want to sort by. **You have to tell it**, and the thing you use to
tell it is a **Comparator**.

A Comparator is simply a rule for deciding order.

```java
record CatalogueItem(String id, String name, double price) { }

List<CatalogueItem> catalogue = new ArrayList<>(List.of(
    new CatalogueItem("P1", "Laptop", 999.0),
    new CatalogueItem("P2", "Phone", 499.0),
    new CatalogueItem("P3", "Tablet", 499.0)
));

// Sort by price, lowest first
catalogue.sort(Comparator.comparing(CatalogueItem::price));
```

Read that last line in plain English: *sort these, comparing by price.*

### What `::` means

`CatalogueItem::price` is a **method reference**. It is shorthand for "take each item and get its
price". The longer way of writing exactly the same thing is a lambda:

```java
Comparator.comparing(item -> item.price())     // the long form
Comparator.comparing(CatalogueItem::price)     // the same thing, shorter
```

The pattern is `ClassName::methodName`, and it means "call this method on each element".

Note that you are not calling the method yourself — there are no brackets after `price`. You are
handing the method over for Java to call later, once per element.

> **Note:** Method references and lambdas are covered fully in a later lesson. For now, recognise
> the shape.

### Chaining and reversing

Two more things you will use constantly.

```java
// Sort by price, then by name where prices are equal
catalogue.sort(Comparator.comparing(CatalogueItem::price)
                         .thenComparing(CatalogueItem::name));

// Highest price first
catalogue.sort(Comparator.comparing(CatalogueItem::price).reversed());
```

`thenComparing()` is the tie-breaker — it is only used when the first comparison comes out equal.
`reversed()` flips the whole order.

These chains read left to right like a sentence: *comparing by price, then by name, reversed.*
That readability is why this style replaced the older way of writing comparators.

> **Note:** `list.sort(...)` and `Collections.sort(list, ...)` both work and do the same thing.
> `list.sort()` is the newer form and is generally preferred in modern code.

---

## 🔵 Optional: Sequenced Collections (Java 21)

> Covered in Lesson 3.5. Included here for reference.

Java 21 added a shared set of methods to collections that have a defined order — `List`, `LinkedHashMap`, and `LinkedHashSet`.

```java
List<String> catalogue = new ArrayList<>(List.of("Laptop", "Phone", "Tablet"));

System.out.println(catalogue.getFirst());   // Laptop
System.out.println(catalogue.getLast());    // Tablet
System.out.println(catalogue.reversed());   // [Tablet, Phone, Laptop]
```

Previously you had to write `catalogue.get(0)` and `catalogue.get(catalogue.size() - 1)`. These methods make the intent clearer and work consistently across ordered collections.

---

## Part 7: Comparison and Summary

| Data Structure | Type | Ordered | Duplicates | Fast at | Slower at |
|----------------|------|---------|------------|---------|-----------|
| Array | Linear | Yes | Yes | Access by index | Fixed size, cannot grow |
| ArrayList | Linear | Yes | Yes | Access by index | Inserting in the middle |
| LinkedList | Linear | Yes | Yes | Adding and removing at the ends | Access by index |
| ArrayDeque | Linear | Yes | Yes | Adding and removing at both ends | Access by index |
| HashMap | Hash-Based | No | Keys unique | Lookup by key | No ordering |
| LinkedHashMap | Hash-Based | Insertion order | Keys unique | Lookup by key, keeps order | Slightly more memory |
| TreeMap | Hash-Based | Sorted by key | Keys unique | Sorted key access | Slower than HashMap |
| HashSet | Hash-Based | No | No | Checking membership | No ordering |
| LinkedHashSet | Hash-Based | Insertion order | No | Membership, keeps order | Slightly more memory |
| TreeSet | Hash-Based | Sorted | No | Sorted unique access | Slower than HashSet |

---

### Choosing a Data Structure

Ask these questions in order:

1. **Do I look things up by a key?** → Use a `Map`.
2. **Do I need every element to be unique?** → Use a `Set`.
3. **Do I need to add and remove from the ends?** → Use `ArrayDeque`.
4. **Otherwise** → Use a `List`, and `ArrayList` unless you have a reason not to.

Then, if you chose a Map or a Set:

- No ordering needed → `HashMap` / `HashSet`
- Insertion order needed → `LinkedHashMap` / `LinkedHashSet`
- Sorted order needed → `TreeMap` / `TreeSet`

**Sensible defaults:** `ArrayList` and `HashMap` cover the large majority of everyday cases. Choose something else only when you have a specific reason.

---

### Key Takeaways
- The Java Collections Framework provides ready-made, tested data structures. Declare by interface, create by class.
- Arrays are fixed size and fast for indexed access. ArrayList adds flexibility on top of an array.
- LinkedList and ArrayDeque are built for adding and removing at the ends. Prefer `ArrayDeque` over `Stack`.
- HashMap and HashSet use hashing for fast lookup, at the cost of ordering.
- LinkedHashMap and LinkedHashSet preserve insertion order. TreeMap and TreeSet keep data sorted.
- Use a `record` for objects stored in Sets or used as Map keys — it compares by value, so duplicates are detected correctly.
- Default to `ArrayList` and `HashMap`, and change only when you have a reason.

---

**End of Lesson 3.4**