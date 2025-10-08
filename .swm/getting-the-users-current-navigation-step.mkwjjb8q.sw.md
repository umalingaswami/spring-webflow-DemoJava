---
title: Getting the user's current navigation step
---
This document describes how the system determines and provides the user's current position within a navigation flow. By identifying the active session and extracting the current step, the application can dynamically guide users and respond to their actions.

The main steps are:

- Identify the user's active flow session
- Determine the current step in the user's navigation
- Provide the current step to the application

```mermaid
flowchart TD
  A[Identify user's active flow session]
  B[Determine current navigation step]
  C[Provide current step to application]
  A --> B --> C
```

# Getting the Current State from the Flow Execution

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
    node1["Identify the user's active flow session"]
    click node1 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:185:200"
    node1 --> node2["Determine the current step in the user's navigation"]
    click node2 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowSessionImpl.java:104:106"
    node2 --> node3["Provide the current step to the application"]
    click node3 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/RequestControlContextImpl.java:109:111"
    node3 --> node4["End"]
    click node4 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/RequestControlContextImpl.java:109:111"


subgraph node1 [getActiveSession]
  sgmain_1_node1{"Is the flow execution active?"}
  click sgmain_1_node1 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:186:193"
  sgmain_1_node1 -->|"Yes"| sgmain_1_node2["Return the active session"]
  click sgmain_1_node2 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:194:194"
  sgmain_1_node1 -->|"No"| sgmain_1_node3{"status == NOT_STARTED?"}
  click sgmain_1_node3 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:187:190"
  sgmain_1_node3 -->|"Yes"| sgmain_1_node4["Exception: No active session, flow not started"]
  click sgmain_1_node4 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:188:189"
  sgmain_1_node3 -->|"No"| sgmain_1_node5["Exception: No active session, flow ended"]
  click sgmain_1_node5 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:191:192"
end

subgraph node2 [getState]
  sgmain_2_node1["Retrieve the current state of the flow execution"]
  click sgmain_2_node1 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/RequestControlContextImpl.java:110:111"
end
```

<SwmSnippet path="/spring-webflow/src/main/java/org/springframework/webflow/engine/impl/RequestControlContextImpl.java" line="109">

---

In `getCurrentState`, we kick off by asking the flow execution for its active session. This is necessary because the session tracks where the user is in the flow, and we need that context before we can figure out the current state. Next, we call getActiveSession to get that session object.

```java
	public StateDefinition getCurrentState() {
		return flowExecution.getActiveSession().getState();
```

---

</SwmSnippet>

## Validating and Fetching the Active Session

```mermaid
%%{init: {"flowchart": {"defaultRenderer": "elk"}} }%%
flowchart TD
  node1{"Is the flow execution active?"}
  click node1 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:186:193"
  node1 -->|"Yes"| node2["Return the active session"]
  click node2 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:194:194"
  node1 -->|"No"| node3{"status == NOT_STARTED?"}
  click node3 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:187:190"
  node3 -->|"Yes"| node4["Exception: No active session, flow not started"]
  click node4 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:188:189"
  node3 -->|"No"| node5["Exception: No active session, flow ended"]
  click node5 openCode "spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java:191:192"
```

<SwmSnippet path="/spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java" line="185">

---

In `getActiveSession`, we first check if the flow execution is active. If it's not, we throw an exception, and the message depends on whether the flow hasn't started or has already ended. This makes it clear why there's no session to return. Next, we call isActive to do this check.

```java
	public FlowSession getActiveSession() {
		if (!isActive()) {
			if (status == FlowExecutionStatus.NOT_STARTED) {
				throw new IllegalStateException(
						"No active FlowSession to access; this FlowExecution has not been started");
			} else {
				throw new IllegalStateException("No active FlowSession to access; this FlowExecution has ended");
			}
		}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java" line="194">

---

After confirming the flow execution is active in `getActiveSession`, we delegate to getActiveSessionInternal to actually fetch the session. This keeps the error handling and the retrieval logic separate.

```java
		return getActiveSessionInternal();
	}
```

---

</SwmSnippet>

## Locating the Most Recent Session

<SwmSnippet path="/spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowExecutionImpl.java" line="555">

---

`getActiveSessionInternal` checks if there are any sessions in flowSessions. If it's empty, we return null. Otherwise, we grab the last session, which is treated as the active one. Next, we use isEmpty to check for the presence of sessions.

```java
	private FlowSessionImpl getActiveSessionInternal() {
		if (flowSessions.isEmpty()) {
			return null;
		}
		return flowSessions.getLast();
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-webflow/src/main/java/org/springframework/webflow/core/collection/LocalParameterMap.java" line="102">

---

`isEmpty` just checks if the parameters map has any entries and returns true if it's empty. That's all there is to it.

```java
	public boolean isEmpty() {
		return parameters.isEmpty();
	}
```

---

</SwmSnippet>

## Extracting the State from the Active Session

<SwmSnippet path="/spring-webflow/src/main/java/org/springframework/webflow/engine/impl/RequestControlContextImpl.java" line="110">

---

After getting the active session in `getCurrentState`, we call getState on that session to fetch the current state definition. This tells us where the user is in the flow.

```java
		return flowExecution.getActiveSession().getState();
	}
```

---

</SwmSnippet>

<SwmSnippet path="/spring-webflow/src/main/java/org/springframework/webflow/engine/impl/FlowSessionImpl.java" line="104">

---

`getState` just returns the current StateDefinition for the session. No extra logic, just hands back the state.

```java
	public StateDefinition getState() {
		return state;
	}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBc3ByaW5nLXdlYmZsb3ctRGVtb0phdmElM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="spring-webflow-DemoJava"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
