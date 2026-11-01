# Gesture Kombat

A browser-based 3D fighting game you control with your body. Your webcam tracks your pose and hands, and your real movements become game actions: pray to cast fire, raise both palms for an ice shield, throw a punch, kick, block, or jump.

**Live demo:** https://funny-choux-d51a43.netlify.app/
**Needs:** a desktop browser with WebGL2, a webcam, and good lighting. A keyboard-and-mouse mode is available if you don't want to use the camera.

> **Privacy:** Camera frames and body landmarks are processed on your device and never uploaded. Only game action names and facing angles are sent to the match server.

## Game modes

| Mode | Description |
|------|-------------|
| **Tutorials** | 14 guided lessons with a 3D demonstration of each move; progress is saved on your device |
| **Human vs AI** | Spar against a server-controlled opponent on easy, normal, or hard |
| **Multiplayer** | Random matchmaking against another queued player |
| **1v1 with Friends** | Create a six-character room code or share an invite link |
| **Live Tracking** | Solo camera and 3D split view to see your pose drive a fighter, with no damage or timer |
| **Classic Play** | Keyboard and mouse, no webcam needed |

Choose between two fighters (Raven and Vortex) and three arenas.

## Controls

| Action | Camera gesture | Keyboard |
|--------|---------------|----------|
| Fire | Praying palms, fingers upright, held briefly | F |
| Ice shield | Both open palms raised, facing the camera | I |
| Lightning | One open hand overhead | E |
| Combo | Charge a prayer, then spread palms | C |
| Punch / double punch | Closed-fist thrust / both fists | J / L |
| Kick | Extend a leg | K |
| Block | Cross your forearms | B |
| Duck | Crouch or bend | S |
| Jump | Jump | W |
| Move | Shift your torso sideways | A / D |
| Walk / run | Alternate knees | A / D |

## How it works

- **Gesture recognition:** MediaPipe Holistic runs in each player's browser. Gesture rules turn pose and hand landmarks into actions, with short holds to reduce jitter. The tracking loop requests up to 60 camera frames per second, depending on hardware.
- **Server-authoritative combat:** The match server decides damage, energy, movement, AI behavior, timeouts, forfeits, and winners. Clients send actions, and replayed input sequences can't apply an attack twice.
- **Match rules:** 90-second rounds, 250 health, and 100 regenerating energy. Attacks have a wind-up, then must connect within a directional hit volume. Fire and combo projectiles travel along a path, and lightning marks a target before striking. Blocking reduces damage and ice absorbs most of it, so moving, crouching, jumping, or shielding can avoid hits.
- **Online play:** The server keeps rooms and the lobby in a shared database with optimistic concurrency. Clients synchronize on a short interval, and a disconnected player stops moving after 700 ms and forfeits after 18 seconds. Match data expires after 15 minutes of inactivity.
- **3D rendering:** Three.js/WebGL with articulated humanoid fighters. The camera tracks your arm and leg joints locally for your own character, while remote players see action animations. Rendering pauses when the tab is hidden and reduces resolution and effects while the camera is active.

## Tech stack

TypeScript, Three.js (WebGL2), MediaPipe Holistic, Node.js, a shared database for match state, and automated tests.

## Run locally

Requires Node 22.13 or later.

```bash
npm run install:ci
npm run db:generate
npm run build
npm run dev
```

Initialize the local database migration before using the game modes. Use the checks below to verify your setup:

```bash
npx tsc --noEmit
node --experimental-strip-types --test tests/combat.test.ts tests/tracking.test.ts tests/characters.test.ts tests/pose3d.test.ts tests/models.test.ts
node tests/online.mjs
```

The project has 55 automated tests, plus type checking, a production build, and multiplayer API checks.

## Known limitations

- This is a hackathon-scale demo. It has no accounts, ranked play, chat, or payments, and it polls the server on a short interval instead of using high-frequency fighting-game netcode. Heavy public traffic would need per-room processes, rate limiting, and abuse controls.
- Gesture thresholds need calibration for each camera. Keep your whole body visible and your lighting good. Leg actions need visible legs, but upper-body tracking works if your hips and feet leave the frame.
- Pose retargeting from a single webcam is approximate, not motion capture, and hidden limbs or full turns can't be reproduced reliably.
- The fighter models are skinned meshes without authored animation clips, so movement is procedural posing plus live tracking. Cloth and armor can deform imperfectly.
- Tracking models load from a CDN, so the first camera start needs an internet connection.
- Latency depends on your network and hardware.

## Credits and contributions

This repository is a fork of [jithendragudapati1-cmd/gesture-kombat](https://github.com/jithendragudapati1-cmd/gesture-kombat).

**My contributions (Shasmith Reddy Dokuri):**

- **Computer vision pipeline:** Built the real-time pipeline that uses MediaPipe Holistic to process webcam pose and hand landmarks and translate physical movements into game commands.
- **Gesture recognition and combat logic:** Developed the gesture recognition, combat logic, and state management, including jitter-reduction logic that turns noisy real-world input into reliable actions.
- **3D rendering and animation:** Built the interactive 3D rendering and procedural character movement with Three.js and WebGL2, integrating tracking, gameplay logic, and UI into one application.
- **Client-server architecture:** Implemented the authoritative game state, multiplayer synchronization, AI-controlled opponents, match lifecycle management, and persistent match data.
- **Testing and reliability:** Tested the system across real camera, lighting, latency, and input conditions, and used automated tests and iterative performance improvements to improve reliability and responsiveness.

Character models were generated with Tripo, and the arena artwork was supplied by the project owner. Check the usage terms of each asset before reusing it.
