# 节点池化Map / PooledMap

PooledMap 是一个高性能的 C++ Map 容器实现，它在保留 `std::map` 功能的同时，显著提升了内存使用效率和访问性能。
可以作为std::map的一个高性能平替

PooledMap is a high-performance C++ Map container implementation that preserves `std::map` functionality while significantly improving memory efficiency and access performance.
It can be used as a high-performance alternative to std::map.

---

## 核心功能 / Features

- **接口兼容 **``** / STL-Compatible Interface**

  - 提供 `operator[]`, `find`, `erase`, `size`, `empty`, `contains` 等常用接口。 Provides common interfaces such as `operator[]`, `find`, `erase`, `size`, `empty`, `contains`.
  - 支持遍历：提供 `for_each` 方法，接收 lambda 代码块操作 key-value。 Supports traversal: provides `for_each` method that accepts a lambda block to operate on key-value pairs.

- **对象池分配 / Object Pooling**

  - 内部节点使用 `SegmentedObjectPool` 进行统一分配。 Internal nodes are allocated using `SegmentedObjectPool`.
  - 杜绝传统 Map 节点反复申请和释放导致的性能开销。 Avoids the performance overhead of repeated allocation and deallocation in traditional Maps.
  - 节点连续分配，增强 CPU 缓存友好性。 Nodes are allocated contiguously to enhance CPU cache friendliness.

- **高性能内存局部性 / Cache-Friendly**

  - 节点在内存中连续存储，提升遍历和查找性能。 Nodes are stored contiguously in memory, improving traversal and lookup performance.
  - 避免散列或链表节点分配带来的内存碎片和低缓存命中率。 Avoids memory fragmentation and low cache hits caused by scattered node allocations.

- **共享对象池 / Shared Pool**

  - 同类型的多个 `PooledMap` 可以共用同一个对象池实例。 Multiple `PooledMap` instances of the same type can share a single object pool instance.
  - 进一步减少内存分配次数，提高性能。 Further reduces memory allocations and improves performance.

- **支持泛型 Key/Value / Generic Key/Value Support**

  - 可用于基本类型（int, float 等）、字符串和自定义类型。 Can be used with basic types (int, float, etc.), strings, and custom types.
  - 对基本类型使用值传递，对复杂类型使用引用传递，实现高效访问。 Uses value passing for basic types and reference passing for complex types to achieve efficient access.

- **红黑树实现 / Balanced Tree**

  - 内部使用红黑树保证操作的 O(log n) 时间复杂度。 Internally uses a red-black tree to guarantee O(log n) operation complexity.
  - 提供稳定有序遍历。 Provides stable and ordered traversal.

---

## 性能优势 / Performance Benefits

1. **内存连续性 / Memory Contiguity**

   - 节点通过对象池连续分配，CPU 缓存命中率显著提高。 Nodes are allocated contiguously through the object pool, significantly improving CPU cache hit rate.

2. **避免重复分配释放 / Avoid Repeated Allocation**

   - 节点复用避免了频繁调用 `new/delete`，降低内存碎片和分配开销。 Node reuse avoids frequent `new/delete` calls, reducing memory fragmentation and allocation overhead.

3. **共享池机制 / Shared Pool Mechanism**

   - 多个同类型 Map 可以共用池，提高大规模 Map 集合的性能。 Multiple maps of the same type can share a pool, improving performance for large map collections.

4. **完美转发 / Efficient Parameter Passing**

   - 支持左值/右值参数的完美转发，基本类型走寄存器传参，提升访问速度。 Supports perfect forwarding of lvalue/rvalue parameters, allowing basic types to be passed via registers for faster access.

---

## 使用示例 / Example

```cpp
#include "PooledMap.hpp"
#include <iostream>

int main() {
    PooledMap<int, std::string> map;
    map[1] = "one";
    map[2] = "two";

    // 遍历 / Traversal
    map.for_each([](const auto& key, auto& value) {
        std::cout << key << ": " << value << std::endl;
    });

    std::cout << "Contains key 2? " << map.contains(2) << std::endl;
    map.erase(1);
    std::cout << "Size after erase: " << map.size() << std::endl;
}
```

与std:map的性能对比测试：
Comparision of performance with that of std::map:

测试用例：
10000 个对象的插入，遍历，删除一半，并再次遍历

Testing Example:
Insertion of 10,000 objects, traversal, deletion of half of them, and traversal again.

```text
==== std::map Test ====
Insert: 1486 ms
Traverse: 472 ms
Erase: 1418 ms
Traverse2: 0 ms
Total: 3377 ms

==== PooledMap Test ====
Insert: 1227 ms
Traverse: 309 ms
Erase: 796 ms
Traverse2: 0 ms
Total: 2333 ms

==== std::map Test ====
Insert: 1534 ms
Traverse: 498 ms
Erase: 1730 ms
Traverse2: 0 ms
Total: 3762 ms

==== PooledMap Test ====
Insert: 1206 ms
Traverse: 457 ms
Erase: 936 ms
Traverse2: 0 ms
Total: 2601 ms
```


PooledMap 适合对性能敏感、节点频繁分配释放的场景，例如游戏开发、金融交易、实时数据处理等。

PooledMap is suitable for performance-critical scenarios where nodes are frequently allocated and deallocated,
such as game development, financial trading, and real-time data processing.

