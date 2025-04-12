# Open Game System (OGS) Specification v1

This repository contains the official specification for the Open Game System (OGS), defining the protocols and requirements for integrating with the OGS ecosystem.

## Table of Contents

- [Introduction](#introduction)
- [Core Specifications](#core-specifications)
  - [Account Linking Protocol](#account-linking-protocol)
  - [Push Notification Protocol](#push-notification-protocol)
  - [TV Casting Protocol](#tv-casting-protocol)
  - [Cloud Rendering Protocol](#cloud-rendering-protocol)
- [Certification Process](#certification-process)
- [Version History](#version-history)

## Introduction

The Open Game System specification defines the protocols and APIs that allow web games to access native capabilities through the OGS ecosystem. This specification is designed to be:

- **Web-First**: Built for games that live primarily on the web
- **Protocol-Based**: Focused on defining clear communication protocols
- **Independent**: Allows games to maintain their own identity and authentication
- **Extensible**: Designed to accommodate future capabilities

The specification is implemented by the OGS platform and SDKs, which follow these protocols to enable cross-platform features.

## Core Specifications

The OGS v1 specification includes core protocols for: Account Linking, Push Notifications, TV Casting, and optional Cloud Rendering.

### Account Linking Protocol

The Account Linking Protocol enables games to link their independent user accounts with the OGS platform, enabling cross-platform features like push notifications while maintaining authentication independence.

#### Protocol Overview

1. Game server requests a link token from the OGS platform
2. User is redirected to the OGS platform's link account page with the link token
3. User authenticates with the OGS platform and confirms the link
4. OGS platform links the accounts and redirects back to the game
5. Game server verifies the link status with the OGS platform

#### HTTP Sequence for Account Linking

```mermaid
sequenceDiagram
    participant Game as Game Server
    participant OGS as OGS Platform
    participant User as User Browser
    
    Game->>OGS: 1. Request Link Token
    OGS-->>Game: 2. Return Link Token & URL
    Game->>User: 3. Redirect to Link URL
    User->>OGS: 4. User Authenticates
    User->>OGS: 5. User Confirms Link
    OGS->>User: 6. Redirect to Game Callback
    Game->>OGS: 7. Verify Link Status
    OGS-->>Game: 8. Return Link Information
```

#### Required API Endpoints

##### Request Link Token (Game Server → OGS Provider)

```http
POST /api/v1/auth/account-link-token HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "gameUserId": "user-123",
  "redirectUrl": "https://yourgame.com/auth/callback"
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "linkToken": "xyz123",
  "expiresAt": "2024-06-30T20:00:00Z",
  "linkUrl": "https://opengame.org/link-account?token=xyz123"
}
```

##### Verify Link Token (Game Server → OGS Provider)

```http
POST /api/v1/auth/verify-link-token HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "token": "xyz123"
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "valid": true,
  "userId": "ogs-user-456",
  "email": "user@example.com",
  "status": "linked",
  "linkedAt": "2024-06-29T15:35:00Z"
}
```

#### Web Auth Token Protocol

The Web Auth Token Protocol enables seamless single sign-on between the OGS platform and games, allowing users to access games through the OGS app without re-authenticating.

##### Protocol Overview

1. OGS app requests a web auth code from the OGS platform
2. OGS app opens the game in a WebView with the code as a parameter
3. Game server verifies the code with the OGS platform
4. OGS platform returns user information
5. Game creates a session for the user

##### HTTP Sequence for Web Auth Token

```mermaid
sequenceDiagram
    participant App as OGS App
    participant OGS as OGS Platform
    participant Game as Game Server
    
    App->>OGS: 1. Request Web Auth Code
    OGS-->>App: 2. Return Web Auth Code
    App->>Game: 3. Open Game URL with Code
    Game->>OGS: 4. Verify Web Auth Code
    OGS-->>Game: 5. Return User Information
    Game->>Game: 6. Create Game Session
```

##### Required API Endpoints

###### Generate Web Auth Code (OGS App → OGS Provider)

```http
POST /api/v1/auth/web-code HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer SESSION_TOKEN
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "code": "xyz123",
  "expiresIn": 300
}
```

###### Verify Web Auth Token (Game Server → OGS Provider)

```http
POST /api/v1/auth/verify-token HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "token": "xyz123"
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "valid": true,
  "ogsUserId": "ogs-user-456",
  "email": "user@example.com",  // Only included if verified
  "isVerified": true
}
```

#### Token Refresh Protocol

The Token Refresh Protocol enables games to refresh expired session tokens using a refresh token.

##### HTTP Sequence for Token Refresh

```mermaid
sequenceDiagram
    participant Game as Game Server
    participant OGS as OGS Platform
    
    Note over Game: Session token expires
    Game->>OGS: 1. Request new token with refresh token
    OGS-->>Game: 2. Return new session token
    Note over Game: Update stored token
```

##### Required API Endpoints

###### Refresh Token (Game Server → OGS Provider)

```http
POST /api/v1/auth/refresh HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Cookie: refresh=refresh-token

{
  "refreshToken": "refresh-token"
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "sessionToken": "new-jwt-token",
  "expiresIn": 900
}
```

### Push Notification Protocol

Push notifications allow games to engage users even when they're not actively playing. The Push Notification Protocol defines how games can send notifications to users through the OGS platform.

#### Protocol Overview

1. Game server sends a notification request to the OGS platform
2. OGS platform verifies the game's API key and the account link
3. OGS platform delivers the notification to the user's device
4. OGS platform returns the delivery status to the game server
5. Game server can later check the notification status (delivered, read, etc.)

#### HTTP Sequence for Sending Notifications

```mermaid
sequenceDiagram
    participant Game as Game Server
    participant OGS as OGS Platform
    participant Device as User Device
    
    Game->>OGS: 1. Send Notification
    OGS->>OGS: 2. Validate Request
    OGS->>Device: 3. Deliver Notification
    OGS-->>Game: 4. Return Delivery Status
    Game->>OGS: 5. Check Notification Status
    OGS-->>Game: 6. Return Updated Status
```

#### Required API Endpoints

##### Send Notification (Game Server → OGS Notification API)

```http
POST /api/v1/notifications/send HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "recipient": {
    "gameUserId": "user-123"
  },
  "notification": {
    "type": "game_invitation",
    "title": "New Invitation",
    "body": "PlayerOne invited you to join Trivia Night!",
    "data": {
      "gameId": "trivia-456",
      "inviterId": "user-789",
      "inviterName": "PlayerOne",
      "gameName": "Trivia Night",
      "expiresAt": "2024-06-30T20:00:00Z"
    },
    "deepLink": "opengame://trivia-jam/join/trivia-456"
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "notification-123",
  "status": "delivered",
  "deliveredAt": "2024-06-29T15:35:00Z"
}
```

##### Get Notification Status (Game Server → OGS Notification API)

```http
GET /api/v1/notifications/status/notification-123 HTTP/1.1
Host: api.opengame.org
Authorization: Bearer GAME_API_KEY
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "notification-123",
  "status": "read",
  "deliveredAt": "2024-06-29T15:35:00Z",
  "readAt": "2024-06-29T15:36:00Z"
}
```

##### Register Device (Game Client → OGS Notification API)

```http
POST /api/v1/notifications/register HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer SESSION_TOKEN

{
  "deviceToken": "device-token-from-fcm-or-apns",
  "platform": "ios", // or "android", "web"
  "topics": ["game_invitation", "turn_notification", "event_reminder"]
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "success": true,
  "deviceId": "device-123"
}
```

### TV Casting Protocol

TV casting allows games to be displayed on larger screens while using mobile devices as controllers. Importantly, **Chromecast support is completely independent** and does NOT require Auth Kit or Notification Kit implementation.

#### Protocol Overview

1. Game server creates a cast session with the OGS platform
2. OGS platform initializes the receiver application on the Chromecast device
3. Game client switches to controller mode
4. Game client sends input events to the OGS platform
5. OGS platform forwards input events to the receiver application
6. When finished, game client ends the cast session

#### HTTP Sequence for Casting

```mermaid
sequenceDiagram
    participant Game as Game Client
    participant OGS as OGS Platform
    participant Cast as Chromecast Device
    
    Game->>OGS: 1. Create Cast Session
    OGS-->>Game: 2. Return Session Info
    OGS->>Cast: 3. Initialize Receiver App
    Game->>Game: 4. Switch to Controller Mode
    Game->>OGS: 5. Send Input Events
    OGS->>Cast: 6. Forward Input Events
    Game->>OGS: 7. End Cast Session
    OGS->>Cast: 8. Terminate Receiver App
```

#### System Architecture

```mermaid
flowchart TD
    subgraph "Mobile Device"
        App[OGS Mobile App]
        WebView[WebView]
        GameUI[Game UI]
        ControllerUI[Controller UI]
        CastSDK[Google Cast SDK]
    end
    
    subgraph "OGS Platform"
        API[OGS API]
        CastService[Cast Service]
        SessionMgr[Session Manager]
    end
    
    subgraph "TV"
        Chromecast[Chromecast Device]
        ReceiverApp[Receiver App]
        GameView[Game View]
    end
    
    GameUI -->|Uses| WebView
    WebView -->|Communicates with| App
    App -->|Uses| CastSDK
    CastSDK -->|Connects to| Chromecast
    
    GameUI -->|Switches to| ControllerUI
    ControllerUI -->|Sends input| API
    API -->|Routes to| CastService
    CastService -->|Manages| SessionMgr
    SessionMgr -->|Sends data to| ReceiverApp
    ReceiverApp -->|Renders| GameView
```

#### Required API Endpoints

##### Create Cast Session (Game Server → OGS Cast API)

```http
POST /api/v1/cast/session HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer GAME_API_KEY

{
  "gameId": "your-game-id",
  "gameUrl": "https://yourgame.com/play?mode=cast",
  "sessionData": {
    "gameState": "initial",
    "players": ["player1", "player2"]
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "sessionId": "cast-session-123",
  "receiverUrl": "https://receiver.opengame.org/cast?session=cast-session-123",
  "status": "created"
}
```

##### Send Input to Cast Session (Game Client → OGS Cast API)

```http
POST /api/v1/cast/input HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer SESSION_TOKEN

{
  "sessionId": "cast-session-123",
  "inputType": "action",
  "inputData": {
    "action": "jump",
    "parameters": {
      "height": 2,
      "direction": "forward"
    }
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "delivered",
  "timestamp": "2024-06-29T15:40:00Z"
}
```

##### Get Cast Session Status (Game Client → OGS Cast API)

```http
GET /api/v1/cast/session/cast-session-123 HTTP/1.1
Host: api.opengame.org
Authorization: Bearer SESSION_TOKEN
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "sessionId": "cast-session-123",
  "status": "active",
  "createdAt": "2024-06-29T15:30:00Z",
  "lastActivityAt": "2024-06-29T15:35:00Z"
}
```

##### End Cast Session (Game Client → OGS Cast API)

```http
DELETE /api/v1/cast/session/cast-session-123 HTTP/1.1
Host: api.opengame.org
Authorization: Bearer SESSION_TOKEN
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "terminated",
  "sessionId": "cast-session-123"
}
```

### Cloud Rendering Protocol

The Cloud Rendering Protocol allows games to offload intensive graphics rendering to powerful cloud servers and stream the output via WebRTC to the client (typically running within the OGS App or a compatible browser). This is ideal for turn-based games or experiences where high visual fidelity is desired and absolute minimal latency is not the primary concern.

#### Protocol Overview

1.  **Client Initialization:** The game client (using `stream-kit`) initializes and optionally informs the OGS backend it's ready for potential streaming.
2.  **Stream Request:** The game client requests a cloud rendering session from the OGS platform, providing the game URL to be rendered and potentially authentication/session data.
3.  **Session Setup:** The OGS Platform (or a dedicated Cloud Rendering Broker service managed by OGS) receives the request, authenticates it, provisions a cloud rendering instance (e.g., a container with a GPU and Puppeteer/equivalent), and instructs it to load the specified `gameUrl`.
4.  **WebRTC Connection:** The OGS Platform facilitates the WebRTC connection negotiation between the Cloud Rendering Instance and the Game Client. The Cloud Rendering Instance captures the rendered game output (video/audio).
5.  **Streaming:** The Cloud Rendering Instance streams the captured video/audio directly to the Game Client via the established WebRTC PeerConnection. The `stream-kit` handles receiving the stream and displaying it within the designated HTML element.
6.  **Input Forwarding (Optional/Future):** For interactive streamed experiences, client-side input events could be captured by the `stream-kit` and sent back to the Cloud Rendering Instance via the WebRTC data channel or through the OGS API. (v1 focuses on non-interactive/display-only streaming initially).
7.  **Session Termination:** The game client requests termination, or the session times out. The OGS Platform tears down the Cloud Rendering Instance and associated resources.

#### HTTP/WebRTC Sequence for Cloud Rendering

```mermaid
sequenceDiagram
    participant Client as Game Client (w/ stream-kit)
    participant OGSPlatform as OGS Platform/Broker
    participant Renderer as Cloud Rendering Instance
    
    Client->>OGSPlatform: 1. Request Stream Session (gameUrl, authData?)
    OGSPlatform->>OGSPlatform: 2. Authenticate & Provision Renderer
    OGSPlatform->>Renderer: 3. Load Game URL
    Renderer->>Renderer: 4. Start Game & Capture Stream
    OGSPlatform->>Client: 5. Signal WebRTC Offer/ICE Candidates (via Broker)
    Client->>OGSPlatform: 6. Signal WebRTC Answer/ICE Candidates (via Broker)
    OGSPlatform->>Renderer: 7. Forward WebRTC Signaling
    Renderer->>Client: 8. Establish Direct WebRTC PeerConnection
    Renderer-->>Client: 9. Stream Video/Audio via WebRTC
    Note right of Client: stream-kit displays video
    Client->>OGSPlatform: 10. Request End Session (Optional)
    OGSPlatform->>Renderer: 11. Terminate Instance
```

#### Required API Endpoints

##### Request Stream Session (Game Client -> OGS Platform/Broker API)

*Note: This API call would likely be abstracted by the `stream-kit`.*

```http
POST /api/v1/stream/session HTTP/1.1
Host: api.opengame.org 
Content-Type: application/json
Authorization: Bearer OGS_SESSION_TOKEN_OR_GAME_API_KEY 

{
  "gameUrl": "https://yourgame.com/path-to-render", // The specific URL the cloud instance should load
  "clientOffer": { ... }, // SDP Offer from client for WebRTC negotiation
  "clientIceCandidates": [ ... ], // ICE candidates from client
  "renderOptions": { // Optional parameters for the rendering instance
    "resolution": "1920x1080",
    "region": "us-central1", // Preferred region for lower latency
    "gpuType": "nvidia-t4" // Optional GPU preference
  },
  "initialGameData": { // Optional data to pass to the game instance in the cloud
     "userToken": "...",
     "sessionId": "..." 
  }
}
```

Response (Success):

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "sessionId": "stream-session-xyz789",
  "status": "negotiating",
  "serverAnswer": { ... }, // SDP Answer from the rendering instance
  "serverIceCandidates": [ ... ] // ICE candidates from rendering instance
}
```

Response (Failure):

```http
HTTP/1.1 4xx/5xx 
Content-Type: application/json

{
  "error": "Failed to provision renderer",
  "message": "No available GPU instances in the requested region." 
}
```

##### End Stream Session (Game Client -> OGS Platform/Broker API)

*Note: This API call would likely be abstracted by the `stream-kit`.*

```http
DELETE /api/v1/stream/session/{sessionId} HTTP/1.1
Host: api.opengame.org
Authorization: Bearer OGS_SESSION_TOKEN_OR_GAME_API_KEY 
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "terminated",
  "sessionId": "stream-session-xyz789"
}
```

##### WebSocket Signaling (Optional Alternative)

For WebRTC negotiation, a WebSocket connection between the Client, OGS Platform, and potentially the Renderer could be used instead of relying solely on HTTP polling/responses for SDP and ICE candidate exchange. This would be more efficient for the real-time nature of negotiation. The specific WebSocket message format would need further definition.

## Certification Process

To certify your game as OGS-compatible:

### 1. Domain Verification

Create a `.well-known/opengame-association.json` file at your domain root:

```json
{
  "appId": "your-game-id",
  "name": "Your Game Name",
  "version": "1.0.0",
  "contact": "developer@yourgame.com",
  "features": ["authentication", "notifications", "chromecast", "streaming"], 
  "apiVersion": "v1",
  "verification": "VERIFICATION_TOKEN"
}
```

The `VERIFICATION_TOKEN` will be provided during certification. Include only the features you've implemented in the features array. The `apiVersion` field should match the version of the OGS API you're implementing (currently "v1").

### 2. Certification Flow

The certification process follows these steps:

1. Create the `.well-known/opengame-association.json` file on your domain
2. Trigger verification by submitting your domain to the OGS developer portal
3. Receive your API key by email after domain verification
4. Integrate the API key in your backend implementation
5. Test your implementation against the OGS platform
6. Submit your game for final review
7. Receive OGS certification after approval

### 3. API Key Generation

After successful domain verification, our automated system will send your API key to the contact email provided in the `.well-known` file:

* Keys are specific to each game and environment
* Store securely, never expose in client-side code
* Use the API key for server-to-server communication with the OGS platform

## SDKs for Implementation

While this specification defines the protocols at the HTTP level, we provide SDKs that implement these protocols to make integration easier:

- **[auth-kit](https://github.com/open-game-system/auth-kit)**: Implementation of the Account Linking Protocol
- **[notification-kit](https://github.com/open-game-system/notification-kit)**: Implementation of the Push Notification Protocol
- **[cast-kit](https://github.com/open-game-system/cast-kit)**: Implementation of the TV Casting Protocol
- **[stream-kit](link-tbd)** (Conceptual): Implementation of the Cloud Rendering Protocol client-side logic

## Integration Overview

```mermaid
flowchart TD
    subgraph "Your Game"
        GameServer[Game Server]
        GameClient[Game Client]
        AuthKit[auth-kit]
        NotificationKit[notification-kit]
        CastKit[cast-kit]
        StreamKit[stream-kit] (Conceptual)
    end
    
    subgraph "OGS System"
        OGSAPI[OGS API]
        CloudRenderer[Cloud Rendering Service]
        OGSApp[OGS Mobile App]
        OGSPlatform[OGS Platform]
    end
    
    GameServer -->|Uses| AuthKit
    GameServer -->|Uses| NotificationKit
    GameClient -->|Uses| CastKit
    GameClient -->|Uses| StreamKit
    
    AuthKit -->|Communicates with| OGSAPI
    NotificationKit -->|Sends notifications via| OGSAPI
    CastKit -->|Communicates with| OGSApp
    StreamKit -->|Requests/Manages Stream via| OGSAPI
    OGSAPI -->|Coordinates| CloudRenderer 
    CloudRenderer -->|WebRTC Stream| StreamKit 
    
    OGSApp -->|Enables native features| OGSPlatform
```

## Future Extensions

The OGS specification is designed to be extensible. Future versions may include:

- **Wallet Integration**: Connecting with web3 wallets
- **In-App Purchases**: Cross-platform payment processing
- **Multiplayer Services**: Matchmaking and real-time communication
- **Achievements & Leaderboards**: Cross-game achievement tracking

## Version History

- **v1.0.0 (March 2025)** - Initial OGS specification release with authentication, push notifications, and Chromecast support. (Cloud Rendering protocol introduced conceptually).

## Contact

- Website: [https://opengame.org](https://opengame.org)
- Email: [hello@opengame.org](mailto:hello@opengame.org)

## License

This specification is licensed under the MIT License. 
