# LAN Op Manager

A Fabric mod for Minecraft 26.2 (Fabric Loader 0.19.3, Java 25) that fixes
operator management on worlds opened to LAN — including worlds tunneled with
tools like E4MC.

## The problem

When you open a singleplayer world to LAN with "Allow Cheats" on:

- **You (the host)** get full command access, but you are never actually
  added to `ops.json`. Your access comes from an internal "is this the host"
  check, not from the real operator permission system.
- **Everyone else who joins** correctly starts with no special permissions
  (permission level 0), same as a brand new player on a dedicated server.
- There is **no `/op` or `/deop` command available** on an integrated/LAN
  server, so you have no way to promote a friend to operator, even though
  you're effectively "in charge" of the world.

This is fine for solo play, but breaks down the moment you want to hand a
trusted friend operator access on a LAN-hosted world (e.g. to help you build,
or to moderate).

## What this mod does

1. **Joining players never get operator access.** This already matches
   vanilla behavior for non-host players, but the mod logs and verifies it so
   you can confirm it in the server log.
2. **Registers real `/op <player>` and `/deop <player>` commands** that write
   to the same `ops.json` file a dedicated server uses. Once someone is
   opped this way, all normal OP-gated commands (`/give`, `/summon`, `/gamemode`,
   etc.) work correctly for them, and stop working the moment they're
   deopped.
3. **Lets the world host bootstrap the first operator.** Since the host is
   never in `ops.json` themselves on a LAN/integrated server, the mod
   specifically allows the recorded world host (via `MinecraftServer#isHost`)
   to run `/op` and `/deop`, in addition to any player who is already a
   genuine operator. Everyone else is rejected with vanilla's usual
   "you do not have permission" error.

On a real dedicated server (`server.properties` + `server.jar`), this mod
does nothing — dedicated servers already have working `/op`/`/deop` and a
real `ops.json` permission model, so the mod steps out of the way entirely.

## Requirements

- Minecraft 26.2 ("Chaos Cubed")
- Fabric Loader 0.19.3+
- Fabric API 0.155.0+26.2 (or newer for 26.2)
- Java 25

## Building

```
./gradlew build
```

The built jar will be in `build/libs/`. Copy `lanop-1.0.0.jar` (not the
`-sources` jar) into your `.minecraft/mods` folder alongside Fabric API.

Before building, double check the `yarn_mappings` value in
`gradle.properties` against the current recommended build listed on
https://fabricmc.net/develop — Yarn mapping builds are published frequently
and the pinned value here may be superseded by the time you build this.

## Usage

1. Load into your world as normal, then use "Open to LAN" (with cheats
   allowed, if you want to be able to run `/op` yourself as host).
2. Have your friend connect via LAN or your E4MC tunnel.
3. As the host, run:
   ```
   /op FriendUsername
   ```
4. Your friend now has full operator access, persisted in `ops.json` inside
   your world's save folder, until you run `/deop FriendUsername`.

Anyone who is not the host and not already an operator will get a permission
error if they try to run `/op`, `/deop`, `/give`, `/summon`, or any other
operator-gated command — exactly like a dedicated server with an empty
`ops.json`.
