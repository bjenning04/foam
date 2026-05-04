# Verto Design

- [Verto Design](#verto-design)
  - [Functionality](#functionality)
  - [Investigation](#investigation)
    - [Verto](#verto)
      - [Verto JavaScript Library](#verto-javascript-library)
      - [Platform Specific Native Implementation](#platform-specific-native-implementation)
      - [React Native](#react-native)
      - [Flutter](#flutter)
      - [Kotlin Multiplatform](#kotlin-multiplatform)
      - [Recommendation](#recommendation)
    - [JSON RPC](#json-rpc)
      - [gRPC](#grpc)
      - [Custom Implementation](#custom-implementation)
      - [Recommendation](#recommendation-1)
    - [WebRTC](#webrtc)
      - [Standard Google](#standard-google)
      - [GetStream](#getstream)
      - [Mesibo](#mesibo)
      - [WebRTC KMP](#webrtc-kmp)
      - [Recommendation](#recommendation-2)
  - [Implementation](#implementation)
    - [High Level Design](#high-level-design)
    - [JsonRpcClient Design](#jsonrpcclient-design)
    - [VertoClient Design](#vertoclient-design)

## Functionality

## Investigation

### Verto

#### [Verto JavaScript Library](https://evoluxbr.github.io/verto-docs/)

Pros:

- Least amount of work since there is an already existing library to consume

Cons:

- Requires a WebView
- Cannot guarantee WebSocket connection on mobile devices (particularly iOS)

#### Platform Specific Native Implementation

Pros:

- Easiest to reimplement for each individual platform

Cons:

- More overall effort to implement and maintain feature parity across all supported platforms

#### [React Native](https://reactnative.dev/)

Pros:

- Does compile some native code for each supported platform

Cons:

- Native code limited to UI
- Business logic remains as JavaScript, accessed via JavaScript bridge
- Since the business logic is still JavaScript, this suffers from the same drawbacks as using the already existing Verto JavaScript library in that we cannot guarantee the WebSocket connection while in the background on mobile devices

#### [Flutter](https://flutter.dev/)

Pros:

- Business logic compiles to native C for each supported platform

Cons:

- No way to embed the native C business logic as a library for other non-Flutter projects, requires Flutter UI
- Would require developers to learn another programming language (Dart)

#### [Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform.html)

Pros:

- Can write all common business logic one time
- Ability to write platform specific implementations for specific features that are not the same for each platform
- Compiles to native code as libraries that are easily consumed by each supported platform
- Ability to write business logic in a programming language that our team already uses (Kotlin)

Cons:

- Some things can be more complex that implementing the same logic natively for an individual platform

#### Recommendation

Recommendation is to reimplement the Verto library using Kotlin Multiplatform.

### JSON RPC

#### [gRPC](https://grpc.io/)

Pros:

- Allows easy generation of both data model and transaction interfaces for multiple programming languages/platforms through a single definition using protocol buffers
- Would eliminate a significant amount of code around WebSocket business logic since this is handled by gRPC itself

Cons:

- Requires HTTP/2, while mod_verto only supports HTTP/1.1

#### Custom Implementation

Pros:

- The Verto JavaScript library uses a relatively simple WebSocket/AJAX JSON RPC implementation that should be fairly easy to convert to Kotlin Multiplatform

Cons:

- More complex implementation
- Significantly more logic to maintain

#### Recommendation

Recommendation is to build a custom JSON RPC implementation using Kotlin Multiplatform. It may be worth looking into replacing mod_verto with a more modern signaling implementation using gRPC at some point in the future.

### WebRTC

#### [Standard Google](https://webrtc.org/)

Pros:

- This is the standard WebRTC implementation
- Can be self-built for multiple platforms

Cons:

- Would have to reimplement the same logic multiple times for each supported platform

#### [GetStream](https://getstream.io/)

Pros:

- Well supported WebRTC library built on top of Google's standard library

Cons:

- Only supports Android
- Some commercial licensing concerns

#### [Mesibo](https://mesibo.com/)

Pros:

- Another well supported WebRTC library built on top of Google's standard library
- Supports multiple platforms, including Android and iOS

Cons:

- Would have to reimplement the same logic multiple times for each supported platform
- Primarily focus towards using their cloud services
- Some commercial licensing concerns

#### [WebRTC KMP](https://github.com/shepeliev/webrtc-kmp)

Pros:

- Basic Kotlin Multiplatform wrapper for the standard Google WebRTC libraries for each platform
- Open source

Cons:

- Maintained by only a few contributors in public Github repo

#### Recommendation

Recommendation is to go with WebRTC KMP, but fork the public open source repository on Github so that we can manage and maintain changes to the library ourselves, and also regularly build the standard Google WebRTC libraries for each supported platform ourselves. This most closely follows Oracle's policies regarding 3rd party libraries, and safeguards us against the security risks inherent to publicly hosted libraries.

## Implementation

### High Level Design

The Voice application for each supported platform shall consume a custom Verto signaling client built as a Kotlin Multiplatform library (verto-client). This verto-client library will in turn consume a custom JSON RPC client also built as a Kotlin Multiplatform library (jsonrpc-client) as well as a fork of the publicly available webrtc-kmp, which is a Kotlin Multiplatform wrapper for Google's standard WebRTC libraries (webrtc-android, webrtc-ios, etc.).

![Verto High Level Design](images/Verto-High-Level.drawio.png)

### JsonRpcClient Design

This library shall allow generic JSON RPC 2.0 communication using either a websocket or AJAX request. It defines the generic data model for performing JSON RPC 2.0 communication, as well as ensuring a persistent websocket connection if a websocket URL is provided.

![JsonRpcClient Design](images/JsonRpcClient.drawio.png)

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
