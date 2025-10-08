---
title: Session Termination in Spring Web Flow
---
# Introduction to Session Termination - End Session

Session termination, or 'end session', is the process of concluding an active flow session within a flow execution in Spring Web Flow. This action removes the current flow session from the execution stack, signaling the completion of the user's interaction within that session.

# Importance of Ending a Session

Properly ending a session is essential for managing the lifecycle of flows. It ensures that resources associated with the completed flow or subflow are released. Additionally, it determines whether the entire flow execution concludes or if control returns to a parent flow session to continue the navigation flow.

# How Session Termination Works

When an EndState is reached in a flow, it triggers the termination of the active flow session. If this session is the root flow session, the entire flow execution ends, marking the logical end of the user's interaction within that flow. Conversely, if the session is a subflow, the flow execution continues by returning control to the parent flow session.

In the case of subflow termination, an ending result event is sent back to the parent flow session. This event allows the parent session to handle the continuation of the flow, enabling complex navigation scenarios where subflows can return results to their callers.

# Example Usage of End Session

The EndState class in Spring Web Flow exemplifies how entering this state ends the current flow session. For subflows, it returns an event to the parent session, which then resumes and continues the flow execution based on the received event.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBc3ByaW5nLXdlYmZsb3ctRGVtb0phdmElM0ElM0F1bWFsaW5nYXN3YW1p" repo-name="spring-webflow-DemoJava"><sup>Powered by [Swimm](https://app.swimm.io/)</sup></SwmMeta>
