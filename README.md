# R-Type

A multiplayer shoot 'em up in C++ with an authoritative, multithreaded UDP game server built on an Entity-Component-System engine and an SFML client.

## About

Epitech team project (2023–2024, third year). The goal was to recreate the classic arcade game R-Type as a networked game: several players connect to the same server and fight waves of enemies together. The server owns the whole game simulation (an in-house ECS); clients send their inputs and render the state they receive.

## My role

[À COMPLÉTER PAR RÉMI]

## Features

**Server**
- Asynchronous UDP networking with standalone **Asio**; the `io_context` runs on a pool of threads (`std::thread::hardware_concurrency()`), with a mutex guarding shared client state.
- Game loop at ~60 Hz on an `asio::steady_timer` (16 ms tick), broadcasting the world state (wave, entity positions, score) to every client.
- Simple text protocol: `START`, `QUIT`, `UP`, `DOWN`, `LEFT`, `RIGHT`, `SHOOT`.
- Room management: player limit, solo mode (`--solo`), starting wave (`--wave 1-10`), configurable port.
- Difficulty (`easy` / `medium` / `hard`) read from `config_game.txt`.

**ECS game engine**
- Entities composed of components: `Position`, `Speed`, `Health` (with shield), `Damages`, `HitBox`, `ShootCooldown`.
- Systems: `PlayerSystem`, `MonsterSystem` (enemy waves), `MissileSystem`, `HitboxSystem` (collisions), `EntitySystem`.
- Several enemy types (minions, kamikazes, elite kamikazes, boss) and power-ups (health, shield, speed).

**Client**
- SFML rendering at 1920×1080 with animated sprites and a parallax scrolling background.
- Main menu and options screen (30 / 60 FPS, sound on / off), music and sound effects.
- Dedicated network thread listening to the server.

**Tooling**
- Cross-platform CMake builds (Linux, macOS, Windows / MinGW); Asio (and SFML on Windows) fetched with `FetchContent`.
- Python integration test that simulates a client against a running server.

## Tech stack

- C++17, CMake
- Asio (standalone) — UDP networking, async I/O, timers
- SFML 2.5 — graphics, window, audio
- `std::thread`, `std::mutex`
- Python 3 for integration tests

## Architecture

```
   ┌─────────────────────┐   inputs (UDP)    ┌────────────────────────────┐
   │ Client (SFML)       │ ────────────────▶ │ Server (Asio, thread pool) │
   │ menu, options,      │                   │  ├─ receive / send         │
   │ rendering, sound    │ ◀──────────────── │  ├─ 16 ms tick timer       │
   └─────────────────────┘   world state     │  └─ ECS systems            │
                                             └────────────────────────────┘

client/   Client (network + scenes), Game (state parsing + rendering), Menu, Options,
          TextureManager, SpriteObject, SoundObject
server/   main (CLI, thread pool), Server (sessions, protocol, game tick), Python tests
ECS/      components/     Position, Speed, Health, Damages, HitBox, ShootCooldown
          SystemManager/  Entity, EntityManager, Player / Monster / Missile / Hitbox systems
assets/   sprites, fonts, music and sound effects
```

## Build & Run

Requirements: CMake 3.10+, a C++17 compiler and SFML 2.5. Asio is downloaded automatically by CMake.

```bash
# Debian / Ubuntu
sudo apt install cmake libsfml-dev libasio-dev
```

Build both binaries (Linux / macOS):

```bash
./build.sh      # builds r-type_server and r-type_client at the repository root
                # and packages them with the assets into r_type_<date>.zip
```

On Windows (MinGW): `build.bat`.

Or build each target manually:

```bash
cmake -S server -B server/build && cmake --build server/build
cmake -S client -B client/build && cmake --build client/build
```

Run from the repository root (the client loads `assets/`, the server reads `config_game.txt`):

```bash
./r-type_server [--serverPort 8000] [--wave 1] [--solo]
./r-type_client [--ip 127.0.0.1] [-p 8000]
```

Integration test (server must be running):

```bash
python3 server/server_test_one_client.py
```
