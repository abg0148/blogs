# Demystifying LSM Trees and SSTables: How Modern Storage Engines Handle Massive Writes

Modern databases like RocksDB, Cassandra, and newer ANN systems like FreshDiskANN rely on **LSM Trees (Log-Structured Merge Trees)** for high-throughput, low-latency writes — while still delivering solid read performance. In this post, we break down how LSM trees work, why they’re used, and the critical components that make them efficient, including **SSTables, compaction**, and **Bloom filters**.

---

## 🔍 What Is an LSM Tree?

**LSM Tree** stands for **Log-Structured Merge Tree**. It's a multi-tiered, write-optimized storage structure designed to:

* Buffer writes in memory
* Flush them to disk in bulk
* Periodically merge and compact files to maintain efficiency

---

## 🧱 Core Components of an LSM Tree

| Component                 | Description                                        |
| ------------------------- | -------------------------------------------------- |
| **MemTable**              | In-memory, sorted write buffer (often a skip list) |
| **Write-Ahead Log (WAL)** | Ensures durability of in-memory writes             |
| **SSTables**              | Immutable, sorted disk files written during flush  |
| **Compaction**            | Merges SSTables, resolves overwrites/deletes       |
| **Bloom Filters**         | Help avoid unnecessary disk reads                  |

---

## 🔀 Write and Flush Workflow

1. Writes go to the **MemTable** (sorted)
2. Written to **WAL** for crash recovery
3. When MemTable is full, it’s **flushed** to disk as an SSTable
4. SSTables accumulate and are later **compacted** into lower levels

---

## 🧸 SSTables: Immutable, Sorted Disk Tables

SSTables (Sorted String Tables) are:

* **Immutable**
* **Sorted by key**
* Contain **index blocks** to allow fast seeks
* May include **Bloom filters** to avoid unnecessary lookups

### Example SSTable:

```
A → 5  
C → 10  
M → 20  
Z → 100
```

Index:

* A → offset 0
* C → offset 20
  ...

---

## 🧠 Compaction: Keeping It Clean

**Compaction** is the process of merging multiple SSTables to:

* Remove overwritten or deleted keys
* Reduce the number of SSTables (lower read amplification)
* Reclaim space

### Example: Overlap Resolution

```
SSTable1: A → 5, B → 10, C → 15  
SSTable2: B → 30, C → [DEL], D → 50
```

After compaction:

```
A → 5  
B → 30  
D → 50
```

---

## 🧱 Multi-Level Storage

LSM trees use a tiered layout:

```
L0 → L1 → L2 → L3 ...
```

* **L0**: Recently flushed data, may overlap
* **L1+**: Sorted, non-overlapping SSTables
* Each level is \~10× the size of the one above

### Compaction Triggers:

* Too many SSTables at L0
* L1 exceeds size threshold
* Stale data (tombstones) needs cleanup

---

## 🔀 Multi-Level Compaction Example

Imagine:

```
L0:
  SSTableA: A → 10, B → 20  
  SSTableB: B → 25, C → [DEL]

L1:
  SSTableX: A → 1, B → 2, C → 3, D → 4
```

Compacted into:

```
A → 10  
B → 25  
D → 4
```

* `C` was deleted
* Latest values win
* Output is sorted

---

## 🌟 Amplification Metrics

| Type                    | What It Means                           | Impact            |
| ----------------------- | --------------------------------------- | ----------------- |
| **Write Amplification** | How many times data is rewritten        | SSD wear, latency |
| **Read Amplification**  | How many places are searched for a read | Query speed       |
| **Space Amplification** | How much extra space is used            | Disk usage        |

---

## 🌸 Bloom Filters: Slashing Read Amplification

Each SSTable has an optional **Bloom filter**, which is:

* Space-efficient
* Tells if a key is *definitely not* in the SSTable
* Reduces unnecessary disk reads

### Read Workflow:

```python
if bloom_filter.might_contain("key"):
    # check SSTable
else:
    # definitely not here → skip SSTable
```

---

## 🧠 A Mental Model: The LSM Tree as a Conveyor Belt

Think of an LSM tree as a **log-based conveyor belt**:

* New data starts at the top (L0, in RAM)
* Periodically slides down to deeper layers (L1 → L2 → ...)
* Compaction cleans and sorts data as it moves
* Bloom filters help you quickly decide whether to reach into a deeper bin

---

## ✅ Summary

| Feature         | LSM Tree Advantage                          |
| --------------- | ------------------------------------------- |
| **Writes**      | Buffered in memory, flushed sequentially    |
| **Reads**       | Optimized via Bloom filters + index blocks  |
| **Deletes**     | Efficient via tombstones and compaction     |
| **Scalability** | Multi-level layout handles billions of keys |

---

## 🐚 Final Thoughts

LSM trees are a powerful design for write-heavy systems — but they come with trade-offs. Techniques like **Bloom filters, leveled compaction**, and **tiered SSTable layout** help balance performance across read, write, and storage efficiency.

Used everywhere from **RocksDB** to **FreshDiskANN**, this structure powers much of today’s high-performance storage infrastructure.
