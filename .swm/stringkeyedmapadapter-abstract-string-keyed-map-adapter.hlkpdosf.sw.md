---
title: 'StringKeyedMapAdapter: Abstract String-Keyed Map Adapter'
---
# Introduction

This document explains the design and implementation choices behind the abstract map adapter for maps keyed by strings. We will cover:

1. Why the class is abstract and how it delegates storage specifics to subclasses.
2. How the adapter implements core Map methods using string-keyed hooks.
3. How iteration and collection views (<SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="75:8:8" line-data="	public Set&lt;String&gt; keySet() {">`keySet`</SwmToken>, values, <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="36:13:13" line-data="	private Set&lt;Entry&lt;String, V&gt;&gt; entrySet;">`entrySet`</SwmToken>) are implemented efficiently.
4. How removal and containment checks are handled through the abstract hooks.

# abstract class with string-keyed hooks

The class is abstract because it does not manage storage directly. Instead, it defines four abstract hook methods that subclasses must implement:

- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="119:7:12" line-data="	protected abstract V getAttribute(String key);">`getAttribute(String key)`</SwmToken> to retrieve a value by key.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="127:7:17" line-data="	protected abstract void setAttribute(String key, V value);">`setAttribute(String key, V value)`</SwmToken> to store a <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="122:28:30" line-data="	 * Hook method that needs to be implemented by concrete subclasses. Puts a key-value pair in the map, overwriting">`key-value`</SwmToken> pair.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="134:7:12" line-data="	protected abstract void removeAttribute(String key);">`removeAttribute(String key)`</SwmToken> to remove a key.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="41:13:15" line-data="		for (Iterator&lt;String&gt; it = getAttributeNames(); it.hasNext();) {">`getAttributeNames()`</SwmToken> to iterate over all keys.

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="112">

---

This design lets subclasses back the map with any storage mechanism while exposing a Map interface keyed by strings. The adapter translates Map operations into calls to these hooks, isolating the storage details.

```java
	// hook methods

	/**
	 * Hook method that needs to be implemented by concrete subclasses. Gets a value associated with a key.
	 * @param key the key to lookup
	 * @return the associated value, or null if none
	 */
	protected abstract V getAttribute(String key);

	/**
	 * Hook method that needs to be implemented by concrete subclasses. Puts a key-value pair in the map, overwriting
	 * any possible earlier value associated with the same key.
	 * @param key the key to associate the value with
	 * @param value the value to associate with the key
	 */
	protected abstract void setAttribute(String key, V value);

	/**
	 * Hook method that needs to be implemented by concrete subclasses. Removes a key and its associated value from the
	 * map.
	 * @param key the key to remove
	 */
	protected abstract void removeAttribute(String key);

	/**
	 * Hook method that needs to be implemented by concrete subclasses. Returns an enumeration listing all keys known to
	 * the map.
	 * @return the key enumeration
	 */
	protected abstract Iterator<String> getAttributeNames();
```

---

</SwmSnippet>

# implementing core Map methods via hooks

The adapter implements standard Map methods by delegating to the hooks. For example:

- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="67:5:10" line-data="	public V get(Object key) {">`get(Object key)`</SwmToken> calls <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="47:3:3" line-data="		return getAttribute(key.toString()) != null;">`getAttribute`</SwmToken> with the string key.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="79:5:15" line-data="	public V put(String key, V value) {">`put(String key, V value)`</SwmToken> calls <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="82:1:1" line-data="		setAttribute(stringKey, value);">`setAttribute`</SwmToken> and returns the previous value.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="92:5:10" line-data="	public V remove(Object key) {">`remove(Object key)`</SwmToken> calls <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="42:1:1" line-data="			removeAttribute(it.next());">`removeAttribute`</SwmToken> and returns the removed value.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="46:5:10" line-data="	public boolean containsKey(Object key) {">`containsKey(Object key)`</SwmToken> checks if <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="47:3:3" line-data="		return getAttribute(key.toString()) != null;">`getAttribute`</SwmToken> returns non-null.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="99:5:7" line-data="	public int size() {">`size()`</SwmToken> counts keys by iterating over <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="41:13:15" line-data="		for (Iterator&lt;String&gt; it = getAttributeNames(); it.hasNext();) {">`getAttributeNames()`</SwmToken>.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="40:5:7" line-data="	public void clear() {">`clear()`</SwmToken> removes all keys by iterating and calling <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="42:1:1" line-data="			removeAttribute(it.next());">`removeAttribute`</SwmToken>.

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="40">

---

This approach keeps the Map interface consistent while relying on the subclass to handle actual data.

```java
	public void clear() {
		for (Iterator<String> it = getAttributeNames(); it.hasNext();) {
			removeAttribute(it.next());
		}
	}

	public boolean containsKey(Object key) {
		return getAttribute(key.toString()) != null;
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="67">

---

&nbsp;

```java
	public V get(Object key) {
		return getAttribute(key.toString());
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="79">

---

&nbsp;

```java
	public V put(String key, V value) {
		String stringKey = String.valueOf(key);
		V previousValue = getAttribute(stringKey);
		setAttribute(stringKey, value);
		return previousValue;
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="92">

---

&nbsp;

```java
	public V remove(Object key) {
		String stringKey = key.toString();
		V retval = getAttribute(stringKey);
		removeAttribute(stringKey);
		return retval;
	}

	public int size() {
		int size = 0;
		for (Iterator<String> it = getAttributeNames(); it.hasNext();) {
			size++;
			it.next();
		}
		return size;
	}
```

---

</SwmSnippet>

# collection views and iterators

The adapter provides <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="75:8:10" line-data="	public Set&lt;String&gt; keySet() {">`keySet()`</SwmToken>, <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="108:8:10" line-data="	public Collection&lt;V&gt; values() {">`values()`</SwmToken>, and <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="63:13:15" line-data="	public Set&lt;Entry&lt;String, V&gt;&gt; entrySet() {">`entrySet()`</SwmToken> views backed by internal classes. These views implement standard Set or Collection interfaces and use iterators that traverse keys via <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="41:13:15" line-data="		for (Iterator&lt;String&gt; it = getAttributeNames(); it.hasNext();) {">`getAttributeNames()`</SwmToken>.

- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="76:24:24" line-data="		return (keySet != null) ? keySet : (keySet = new KeySet());">`KeySet`</SwmToken> returns an iterator over keys.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="109:24:24" line-data="		return (values != null) ? values : (values = new Values());">`Values`</SwmToken> returns an iterator that fetches values by key.
- <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="229:5:5" line-data="	private class EntrySet extends AbstractSet&lt;Entry&lt;String, V&gt;&gt; {">`EntrySet`</SwmToken> returns an iterator producing Map.Entry objects for each key.

These internal classes extend an abstract set that delegates size, emptiness, and clear operations back to the adapter, ensuring consistency.

Iterators extend an abstract key iterator that wraps the key enumeration and supports removal by delegating to the adapter's remove method.

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="75">

---

This design avoids duplicating storage and keeps views synchronized with the underlying data.

```java
	public Set<String> keySet() {
		return (keySet != null) ? keySet : (keySet = new KeySet());
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="108">

---

&nbsp;

```java
	public Collection<V> values() {
		return (values != null) ? values : (values = new Values());
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="159">

---

&nbsp;

```java
	private class KeySet extends AbstractSet<String> {
		public Iterator<String> iterator() {
			return new KeyIterator();
		}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="200">

---

&nbsp;

```java
	private class Values extends AbstractSet<V> {
		public Iterator<V> iterator() {
			return new ValuesIterator();
		}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="229">

---

&nbsp;

```java
	private class EntrySet extends AbstractSet<Entry<String, V>> {
		public Iterator<Entry<String, V>> iterator() {
			return new EntryIterator();
		}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="173">

---

&nbsp;

```java
	private abstract class AbstractKeyIterator {
		private final Iterator<String> it = getAttributeNames();

		private String currentKey;

		public void remove() {
			if (currentKey == null) {
				throw new NoSuchElementException("You must call next() at least once");
			}
			StringKeyedMapAdapter.this.remove(currentKey);
		}

		public boolean hasNext() {
			return it.hasNext();
		}

		protected String nextKey() {
			return currentKey = it.next();
		}
	}

	private class KeyIterator extends AbstractKeyIterator implements Iterator<String> {
		public String next() {
			return nextKey();
		}
	}
```

---

</SwmSnippet>

# containment and removal in views

The collection views implement <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="164:5:5" line-data="		public boolean contains(Object o) {">`contains`</SwmToken> and <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="92:5:5" line-data="	public V remove(Object key) {">`remove`</SwmToken> by leveraging the adapter's core methods:

- `KeySet.contains` calls <SwmToken path="spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" pos="46:5:5" line-data="	public boolean containsKey(Object key) {">`containsKey`</SwmToken>.
- `Values.contains` iterates values and compares.
- `EntrySet.contains` checks if the map contains an entry with matching key and value.
- Removal methods remove the corresponding key or entry from the adapter.

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="164">

---

This ensures that changes to views reflect in the underlying map and vice versa.

```java
		public boolean contains(Object o) {
			return StringKeyedMapAdapter.this.containsKey(o);
		}

		public boolean remove(Object o) {
			return StringKeyedMapAdapter.this.remove(o) != null;
		}
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="205">

---

&nbsp;

```java
		public boolean contains(Object o) {
			return StringKeyedMapAdapter.this.containsValue(o);
		}

		public boolean remove(Object o) {
			if (o == null) {
				return false;
			}
			for (Iterator<V> it = iterator(); it.hasNext();) {
				if (o.equals(it.next())) {
					it.remove();
					return true;
				}
			}
			return false;
		}
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-binding/src/main/java/org/springframework/binding/collection/StringKeyedMapAdapter.java" line="234">

---

&nbsp;

```java
		public boolean contains(Object o) {
			Entry<String, V> entry = getAsEntry(o);
			if (entry == null || entry.getKey() == null || entry.getValue() == null) {
				return false;
			}
			V valueFromThisMap = StringKeyedMapAdapter.this.get(entry.getKey());
			return entry.getValue().equals(valueFromThisMap);
		}

		public boolean remove(Object o) {
			Entry<String, V> entry = getAsEntry(o);
			if (entry == null || entry.getKey() == null || entry.getValue() == null) {
				return false;
			}
			V valueFromThisMap = StringKeyedMapAdapter.this.get(entry.getKey());
			if (!entry.getValue().equals(valueFromThisMap)) {
				return false;
			}
			return StringKeyedMapAdapter.this.remove(entry.getKey()) != null;
		}

		@SuppressWarnings("unchecked")
		private Entry<String, V> getAsEntry(Object o) {
			if (o instanceof Entry) {
				return (Entry<String, V>) o;
			}
			return null;
		}
	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBc3ByaW5nLXdlYmZsb3ctRGVtb0phdmElM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="spring-webflow-DemoJava"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
