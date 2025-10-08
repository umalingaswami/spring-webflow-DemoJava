---
title: SharedMapDecorator Class Overview
---
# Introduction

This document explains the rationale behind the implementation of the <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" pos="42:3:3" line-data="	public SharedMapDecorator(Map&lt;K, V&gt; map) {">`SharedMapDecorator`</SwmToken> class. We will cover:

1. Why the class wraps a Map and how it delegates operations.
2. How synchronization is handled via the mutex object.
3. The purpose of the <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" pos="102:5:5" line-data="	public String toString() {">`toString`</SwmToken> method override.

# wrapping and delegating to a target map

<SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" pos="42:3:3" line-data="	public SharedMapDecorator(Map&lt;K, V&gt; map) {">`SharedMapDecorator`</SwmToken> is designed as a wrapper around a standard Map instance. It holds a reference to the wrapped map and delegates all Map interface methods directly to this underlying map. This approach allows it to add behavior or metadata without changing the original map's implementation or usage.

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" line="38">

---

The constructor takes the target map and stores it for delegation:

```java
	/**
	 * Creates a new shared map decorator.
	 * @param map the map that is shared by multiple threads, to be synced
	 */
	public SharedMapDecorator(Map<K, V> map) {
		this.map = map;
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" line="46">

---

All the core Map operations like put, get, remove, size, and others simply call the corresponding method on the wrapped map:

```java
	// implementing Map

	public void clear() {
		map.clear();
	}

	public boolean containsKey(Object key) {
		return map.containsKey(key);
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" line="64">

---

&nbsp;

```java
	public V get(Object key) {
		return map.get(key);
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" line="76">

---

&nbsp;

```java
	public V put(K key, V value) {
		return map.put(key, value);
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" line="84">

---

&nbsp;

```java
	public V remove(Object key) {
		return map.remove(key);
	}

	public int size() {
		return map.size();
	}
```

---

</SwmSnippet>

This delegation keeps the decorator lightweight and focused on its additional responsibilities rather than re-implementing map logic.

# synchronization via mutex object

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" line="96">

---

The class implements the <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" pos="96:5:5" line-data="	// implementing SharedMap">`SharedMap`</SwmToken> interface, which requires providing a mutex object for synchronization. By default, this mutex is the wrapped map itself:

```java
	// implementing SharedMap

	public Object getMutex() {
		return map;
	}
```

---

</SwmSnippet>

This design allows clients to synchronize on the mutex when accessing the map concurrently. Subclasses can override <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" pos="98:5:7" line-data="	public Object getMutex() {">`getMutex()`</SwmToken> to provide a different synchronization object if needed, offering flexibility in concurrency control.

# enhanced <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" pos="102:5:5" line-data="	public String toString() {">`toString`</SwmToken> for debugging

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" line="102">

---

The <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/SharedMapDecorator.java" pos="102:5:5" line-data="	public String toString() {">`toString`</SwmToken> method is overridden to include both the wrapped map and the mutex in its output:

```java
	public String toString() {
		return new ToStringCreator(this).append("map", map).append("mutex", getMutex()).toString();
	}
}
```

---

</SwmSnippet>

This helps during debugging by showing the internal state of the decorator and the synchronization object it uses, making it easier to understand the concurrency context and contents of the map at runtime.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBc3ByaW5nLXdlYmZsb3ctRGVtb0phdmElM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="spring-webflow-DemoJava"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
