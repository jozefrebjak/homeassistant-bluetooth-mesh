# About this fork

Fork of [dominikberse/homeassistant-bluetooth-mesh](https://github.com/dominikberse/homeassistant-bluetooth-mesh),
used to drive Nordlux Oja Smart lights from Home Assistant over Bluetooth SIG Mesh.

Upstream has had no commits since August 2023. This fork exists to keep the
project usable, not to take it over.

## What is different

| Change | Why |
|---|---|
| `docker/Dockerfile` copies the repository instead of cloning upstream | The upstream image never contained the code of the checkout it was built from, which made a fork pointless |
| Build context moved to the repository root | Needed for the change above |
| `.github/workflows/docker-arm64.yml` | Builds a `linux/arm64` image on a native arm64 runner and pushes it to GHCR |
| `docker/docker-compose.ghcr.yaml` | Runs the prebuilt image instead of compiling on the target |

## Why the prebuilt image matters

The image compiles `ell`, `json-c`, BlueZ 5.66 and the pinned Python wheels
(including numpy). On a Raspberry Pi 3B+ that is a multi-hour build that also
needs the swap file raised past the 1 GB of RAM the board has. On a GitHub
arm64 runner it is a normal build, and the Pi only pulls the result.

## Running it on the Pi

The host must hand the Bluetooth controller over to the container, which starts
its own `bluetooth-meshd`:

```bash
systemctl disable --now bluetooth bluetooth-mesh

cd docker
docker compose -f docker-compose.ghcr.yaml pull
docker compose -f docker-compose.ghcr.yaml up -d
```

Provisioning and configuration are unchanged from upstream - see the main
[README](README.md).

## Known issues, not yet fixed

Found by reading the sources; none of them are addressed here yet, because this
branch is only about getting a buildable image.

| Where | Issue |
|---|---|
| `gateway/mqtt/messenger.py` | imports `asyncio_mqtt`, renamed upstream to `aiomqtt` and unmaintained under the old name |
| `gateway/mqtt/bridges/light.py` | publishes `color_temp` in mireds; Home Assistant moved to Kelvin |
| `gateway/mqtt/bridges/light.py` | announces `color_mode: true` but the state message never carries a `color_mode` field |
| `gateway/mqtt/bridges/light.py` | `brightness_scale` hardcoded to 50 although mesh Lightness is 16-bit |
| `gateway/mqtt/bridges/light.py` | discovery payload has no `device` block, so entities arrive in HA without a device |
| `gateway/mqtt/bridges/light.py` | no availability / LWT topic, so HA cannot tell that the gateway died |
| `gateway/mqtt/bridges/light.py` | stray `from sre_constants import BIGCHARSET`, unused and from a deprecated module |
| `requirements.txt` | pins from 2022 (`cryptography==3.3.2`, `numpy==1.23.3`) force the `python:3.10-bullseye` base |
