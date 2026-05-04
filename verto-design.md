# Verto Design

- [Verto Design](#verto-design)
  - [Functionality](#functionality)
  - [Implementation](#implementation)
    - [High Level Design](#high-level-design)
    - [JsonRpcClient Design](#jsonrpcclient-design)
    - [VertoClient Design](#vertoclient-design)

## Functionality

## Implementation

### High Level Design

The Voice application for each supported platform shall consume a custom Verto signaling client built as a Kotlin Multiplatform library (verto-client). This verto-client library will in turn consume a custom JSON RPC client also built as a Kotlin Multiplatform library (jsonrpc-client) as well as a fork of the publicly available webrtc-kmp, which is a Kotlin Multiplatform wrapper for Google's standard WebRTC libraries (webrtc-android, webrtc-ios, etc.).

### JsonRpcClient Design

This library shall allow generic JSON RPC 2.0 communication using either a websocket or AJAX request. It defines the generic data model for performing JSON RPC 2.0 communication, as well as ensuring a persistent websocket connection if a websocket URL is provided.

![JsonRpcClient Design](images/JsonRpcClient.drawio.png)
```mermaid
---
displayMode: compact
---
classDiagram
    JsonRpcClient --> JsonRpcObserver
    JsonRpcClient --> JsonRpcMessage
    JsonRpcMessage <|-- JsonRpcRequest
    JsonRpcMessage <|-- JsonRpcResponse
    JsonRpcRequest <|-- LoginRequest

    class JsonRpcClient {
        +
    }
    class JsonRpcMessage
    class JsonRpcObserver
    class JsonRpcRequest
    class JsonRpcResponse
    class LoginRequest
```

### VertoClient Design

This is the detailed design and service contract for the Verto signaling client. It includes all of the standard functions necessary for voice calling, including video and screen sharing. The VertoClient will track a list of Dialog objects, which are responsible for maintaining the link between individual calls to their respective WebRTC peer connections, and also for updating WebRTC tracks/media as necessary.

**Note**: All public classes accessible by the consumer are highlighted in blue.

![VertoClient Detailed Design](images/VertoClient.drawio.png)

Both the VertoClient and associated Dialogs will make use of the below outlined internal JsonRpcRequest implementations for all necessary communication with mod_verto in FreeSwitch.

![Verto Internal Data Model](images/Verto-Internal-Data-Model.drawio.png)

Each individual call dialog has a number of states that it can flow through as part of a call workflow. Below is a state diagram detailing each state and its corresponding valid next states.

![Verto Dialog State Flow](images/Verto-Dialog-State-Flow.drawio.png)

Below is a sequence diagram showing Verto signaling and call state for a basic scenario.

![Basic Call State Sequence](images/Basic-Call-State-Sequence.drawio.png)
