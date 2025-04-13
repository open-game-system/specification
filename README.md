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

The Cloud Rendering Protocol enables games to offload intensive graphics rendering to powerful cloud servers and stream the output via WebRTC to the client. Unlike other cloud gaming solutions that require streaming the entire game, OGS Stream Kit allows selective cloud rendering of specific graphics-intensive components while maintaining local rendering for UI elements, creating a hybrid rendering approach ideal for web games.

This protocol is especially valuable for:
- Turn-based games requiring high-fidelity 3D graphics
- Games that need to support low-powered devices without compromising visual quality
- Experiences where consistent visual quality across all devices is important
- Games with graphically complex scenes that would benefit from powerful GPU rendering

#### Stream Kit Architecture

The Stream Kit is built on a component-based architecture that allows developers to integrate cloud rendering selectively within their existing web games:

1. **StreamCanvas Component**: A client-side component that displays cloud-rendered content within your web UI
2. **Render Routes**: Standalone web pages optimized for cloud rendering that contain only the graphics-intensive elements
3. **Stream Session Management**: Handles negotiation between client and cloud rendering service
4. **WebRTC Pipeline**: Delivers low-latency video and audio streams from cloud to client

This architecture allows developers to:
- Render only specific components in the cloud rather than the entire application
- Compose multiple StreamCanvas instances in a single UI
- Maintain responsive local UI elements while offloading graphically intensive rendering
- Seamlessly switch between cloud and local rendering based on device capabilities

#### Implementation Strategy

Stream Kit is designed for progressive enhancement of web games:

1. **Standalone Route Creation**: Create dedicated routes in your web application that render only the graphics-intensive elements (3D scenes, complex visualizations, etc.)
2. **Component Integration**: Integrate the StreamCanvas component from stream-kit into your main UI where these elements should appear
3. **Conditional Rendering**: Implement logic to conditionally use cloud rendering based on device capabilities, user preferences, or specific high-fidelity needs
4. **Composition**: Combine multiple StreamCanvas instances to create rich, interactive experiences with different cloud-rendered views

This approach maintains your game's web-first nature while enhancing it with powerful cloud rendering capabilities.

#### Protocol Overview

1. **Client Initialization:** The game client (using `stream-kit`) initializes a StreamCanvas component with a render URL and options.
2. **Stream Request:** The StreamCanvas component requests a cloud rendering session from the OGS platform.
3. **Session Setup:** The OGS Platform authenticates the request, provisions a cloud rendering instance, and instructs it to load the specified render route URL.
4. **WebRTC Connection:** The OGS Platform facilitates WebRTC connection negotiation between the Cloud Rendering Instance and the Game Client.
5. **Streaming:** The Cloud Rendering Instance streams the captured video/audio directly to the Game Client via the established WebRTC connection.
6. **Local Input Handling:** The client can handle user interactions locally while displaying the cloud-rendered content.
7. **Input Forwarding (Optional):** For interactive streamed experiences, client input events can be forwarded to the cloud renderer.
8. **Session Termination:** The game client requests termination when done, and the OGS Platform tears down the Cloud Rendering Instance.

#### HTTP/WebRTC Sequence for Cloud Rendering

```mermaid
sequenceDiagram
    participant Client as Game Client (w/ stream-kit)
    participant OGSPlatform as OGS Platform/Broker
    participant Renderer as Cloud Rendering Instance
    
    Client->>OGSPlatform: 1. Initialize StreamCanvas (renderUrl, options)
    OGSPlatform->>OGSPlatform: 2. Authenticate & Provision Renderer
    OGSPlatform->>Renderer: 3. Load Render Route URL
    Renderer->>Renderer: 4. Start Rendering & Prepare Stream
    OGSPlatform->>Client: 5. Return Session Info & Signaling Data
    Client->>OGSPlatform: 6. Complete WebRTC Negotiation
    OGSPlatform->>Renderer: 7. Forward Negotiation Data
    Renderer->>Client: 8. Establish Direct WebRTC Connection
    Renderer-->>Client: 9. Stream Video/Audio via WebRTC
    Note over Client: StreamCanvas displays video
    Client->>Client: 10. Handle User Interaction Locally
    Client->>OGSPlatform: 11. Send Input Events (Optional)
    OGSPlatform->>Renderer: 12. Forward Input Events
    Client->>OGSPlatform: 13. Request End Session (When Done)
    OGSPlatform->>Renderer: 14. Terminate Instance
```

#### Stream Kit Integration Flow

```mermaid
flowchart TB
    subgraph "Your Web Game"
        MainUI[Main Game UI]
        RenderRoutes[Standalone Render Routes]
        StreamKit[stream-kit SDK]
    end
    
    subgraph "OGS Cloud Infrastructure"
        API[OGS Stream API]
        Provisioner[Renderer Provisioner]
        CloudGPU[Cloud GPU Instances]
        WebRTC[WebRTC Service]
    end
    
    subgraph "End User"
        Device[User Device]
        LocalUI[Local UI Elements]
        StreamCanvas[StreamCanvas]
    end
    
    MainUI -->|Creates| RenderRoutes
    MainUI -->|Uses| StreamKit
    StreamKit -->|Requests Stream| API
    API -->|Provisions| Provisioner
    Provisioner -->|Allocates| CloudGPU
    CloudGPU -->|Loads| RenderRoutes
    CloudGPU -->|Negotiates via| WebRTC
    WebRTC -->|Connects to| StreamKit
    StreamKit -->|Displays in| StreamCanvas
    Device -->|Renders| LocalUI
    LocalUI -->|Composes with| StreamCanvas
    
    style StreamKit fill:#9333EA,stroke:#d8b4fe,color:#ffffff
    style StreamCanvas fill:#9333EA,stroke:#d8b4fe,color:#ffffff
    style CloudGPU fill:#2563eb,stroke:#93c5fd,color:#ffffff
```

#### Stream Kit API Reference

##### StreamCanvas Component Properties

The `StreamCanvas` component is the primary interface for integrating cloud rendering:

```typescript
interface StreamCanvasProps {
  // URL of the standalone render route to be cloud-rendered
  url: string;
  
  // Styling options for the canvas
  className?: string;
  style?: React.CSSProperties;
  
  // Rendering preferences
  renderOptions?: {
    resolution?: "720p" | "1080p" | "1440p" | "4k" | string;
    targetFps?: number;
    quality?: "low" | "medium" | "high" | "ultra";
    priority?: "latency" | "quality";
    region?: string; // Preferred cloud region for reduced latency
  };
  
  // Initial data to pass to the render route
  initialData?: Record<string, any>;
  
  // Callbacks
  onReady?: () => void;
  onError?: (error: Error) => void;
  onStateChange?: (state: StreamState) => void;
  
  // Input handling
  handleInputLocally?: boolean; // Whether to handle input events locally
  forwardInput?: boolean; // Whether to forward input events to cloud
}

interface StreamState {
  status: "initializing" | "connecting" | "streaming" | "reconnecting" | "error" | "ended";
  latency?: number; // Estimated round-trip latency in ms
  resolution?: string; // Actual streaming resolution
  fps?: number; // Current frames per second
  errorMessage?: string;
}
```

#### Required API Endpoints

##### Request Stream Session (Client SDK → OGS Platform)

```http
POST /api/v1/stream/session HTTP/1.1
Host: api.opengame.org 
Content-Type: application/json
Authorization: Bearer OGS_SESSION_TOKEN_OR_GAME_API_KEY 

{
  "renderUrl": "https://yourgame.com/path-to-render",
  "clientId": "unique-client-identifier",
  "renderOptions": {
    "resolution": "1920x1080",
    "targetFps": 60,
    "quality": "high",
    "priority": "latency",
    "region": "us-central1"
  },
  "initialData": {
    "sceneId": "world-map-01",
    "userToken": "user-auth-token",
    "viewParameters": {
      "camera": { "x": 0, "y": 10, "z": -5 },
      "target": { "x": 0, "y": 0, "z": 0 }
    }
  },
  "webRtcConfig": {
    "iceServers": [
      { "urls": "stun:stun.l.google.com:19302" }
    ],
    "sdpSemantics": "unified-plan"
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "sessionId": "stream-session-xyz789",
  "status": "initializing",
  "rendererId": "renderer-abc123",
  "signalingUrl": "wss://signaling.opengame.org/stream/xyz789",
  "iceServers": [
    { "urls": "stun:stun.l.google.com:19302" },
    { 
      "urls": "turn:turn.opengame.org:3478", 
      "username": "username", 
      "credential": "password" 
    }
  ],
  "estimatedStartTime": 500, // ms until stream should begin
  "region": "us-central1"
}
```

##### Send Input to Cloud Renderer (Client SDK → OGS Platform)

```http
POST /api/v1/stream/session/{sessionId}/input HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer OGS_SESSION_TOKEN_OR_GAME_API_KEY 

{
  "type": "interaction",
  "timestamp": 1627845292123,
  "data": {
    "action": "select",
    "position": { "x": 250, "y": 300 },
    "entityId": "character-5",
    "additionalData": {
      "pressure": 0.8
    }
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "received": true,
  "timestamp": 1627845292150,
  "latency": 27 // ms between client timestamp and server receipt
}
```

##### Update Stream Parameters (Client SDK → OGS Platform)

```http
PATCH /api/v1/stream/session/{sessionId} HTTP/1.1
Host: api.opengame.org
Content-Type: application/json
Authorization: Bearer OGS_SESSION_TOKEN_OR_GAME_API_KEY 

{
  "renderOptions": {
    "resolution": "1280x720", // Downgrade resolution if network conditions change
    "targetFps": 30
  },
  "sceneData": {
    "viewParameters": {
      "camera": { "x": 10, "y": 15, "z": -8 },
      "target": { "x": 5, "y": 0, "z": 2 }
    }
  }
}
```

Response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "accepted": true,
  "applied": {
    "resolution": "1280x720",
    "targetFps": 30
  },
  "effective": {
    "actualFps": 32,
    "actualBitrate": 2500000 // bits per second
  }
}
```

##### End Stream Session (Client SDK → OGS Platform)

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
  "sessionId": "stream-session-xyz789",
  "usage": {
    "duration": 1200, // seconds
    "dataTransferred": 450000000, // bytes
    "renderUnitTime": 20 // minutes of GPU time used
  }
}
```

##### WebSocket Signaling (Optional Alternative)

For WebRTC negotiation, a WebSocket connection between the Client, OGS Platform, and potentially the Renderer could be used instead of relying solely on HTTP polling/responses for SDP and ICE candidate exchange. This would be more efficient for the real-time nature of negotiation. The specific WebSocket message format would need further definition.

#### Stream Kit Integration Examples

##### Basic Integration (React)

```jsx
import { StreamCanvas } from '@open-game-system/stream-kit';

function GameView() {
  return (
    <div className="game-container">
      <header className="game-hud">
        {/* Locally rendered UI elements */}
        <div className="score">Score: 1250</div>
        <div className="health-bar">Health: 85%</div>
      </header>
      
      <main className="game-view">
        {/* Cloud-rendered 3D scene */}
        <StreamCanvas 
          url="https://yourgame.com/render/world-scene"
          className="w-full h-full"
          renderOptions={{
            resolution: "1080p",
            quality: "high"
          }}
          onStateChange={(state) => console.log("Stream state:", state)}
        />
      </main>
      
      <footer className="game-controls">
        {/* Locally rendered control UI */}
        <button>Inventory</button>
        <button>Map</button>
      </footer>
    </div>
  );
}
```

##### Multiple View Integration

```jsx
import { StreamCanvas } from '@open-game-system/stream-kit';
import { useState } from 'react';
import * as Tabs from '@radix-ui/react-tabs';

function GameWithMultipleViews() {
  const [activeView, setActiveView] = useState('world');
  
  return (
    <div className="game-container">
      <Tabs.Root value={activeView} onValueChange={setActiveView}>
        <Tabs.List className="view-selector">
          <Tabs.Trigger value="world">World</Tabs.Trigger>
          <Tabs.Trigger value="map">Map</Tabs.Trigger>
          <Tabs.Trigger value="character">Character</Tabs.Trigger>
        </Tabs.List>
        
        <Tabs.Content value="world" className="view-content">
          <StreamCanvas 
            url="https://yourgame.com/render/world-view"
            className="w-full h-full"
          />
        </Tabs.Content>
        
        <Tabs.Content value="map" className="view-content">
          <StreamCanvas 
            url="https://yourgame.com/render/map-view"
            className="w-full h-full"
          />
        </Tabs.Content>
        
        <Tabs.Content value="character" className="view-content">
          <StreamCanvas 
            url="https://yourgame.com/render/character-view"
            className="w-full h-full"
          />
        </Tabs.Content>
      </Tabs.Root>
      
      <div className="game-controls">
        {/* Common controls that work across all views */}
        <button>Inventory</button>
        <button>Menu</button>
      </div>
    </div>
  );
}
```

##### Conditional Cloud Rendering

```jsx
import { StreamCanvas } from '@open-game-system/stream-kit';
import { useEffect, useState } from 'react';
import { LocalRenderer } from './components/LocalRenderer';
import { detectDeviceCapabilities } from './utils/deviceDetection';

function AdaptiveGameView() {
  const [useCloudRendering, setUseCloudRendering] = useState(false);
  
  useEffect(() => {
    // Detect if the device would benefit from cloud rendering
    const deviceCapabilities = detectDeviceCapabilities();
    
    setUseCloudRendering(
      deviceCapabilities.gpuTier < 2 || // Low-end GPU
      deviceCapabilities.isMobile ||    // Mobile device
      deviceCapabilities.preferCloudRendering // User preference
    );
  }, []);
  
  return (
    <div className="game-scene">
      {useCloudRendering ? (
        <StreamCanvas 
          url="https://yourgame.com/render/high-quality-scene"
          className="w-full h-full"
          renderOptions={{ quality: "high" }}
        />
      ) : (
        <LocalRenderer 
          scene="high-quality-scene"
          className="w-full h-full"
          quality={deviceCapabilities.gpuTier >= 3 ? "ultra" : "medium"}
        />
      )}
      
      <div className="overlay-controls">
        <button onClick={() => setUseCloudRendering(!useCloudRendering)}>
          Toggle Cloud Rendering
        </button>
      </div>
    </div>
  );
}
```

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

The following SDKs implement the OGS protocols to simplify integration:

- **[auth-kit](https://github.com/open-game-system/auth-kit)**: Implementation of the Account Linking Protocol
- **[notification-kit](https://github.com/open-game-system/notification-kit)**: Implementation of the Push Notification Protocol
- **[cast-kit](https://github.com/open-game-system/cast-kit)**: Implementation of the TV Casting Protocol
- **[stream-kit](https://github.com/open-game-system/stream-kit)**: Implementation of the Cloud Rendering Protocol with components for integrating cloud-rendered graphics into web games

Each SDK can be used independently, allowing you to implement only the features your game needs.

## Integration Overview

```mermaid
flowchart TD
    subgraph "Your Game"
        GameServer[Game Server]
        GameClient[Game Client]
        AuthKit[auth-kit]
        NotificationKit[notification-kit]
        CastKit[cast-kit]
        StreamKit[stream-kit]
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

- **v1.1.0 (Planned Q4 2024)** - Addition of formalized Stream Kit specification and cloud rendering capabilities
- **v1.0.0 (March 2024)** - Initial OGS specification release with authentication, push notifications, and Chromecast support

## Contact

- Website: [https://opengame.org](https://opengame.org)
- Email: [hello@opengame.org](mailto:hello@opengame.org)

## License

This specification is licensed under the MIT License. 
