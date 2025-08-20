# Pi-Star Docker Containers

Multi-architecture Docker containers for MMDVMHost and related Pi-Star software.

## Usage

```bash
docker run -d \
  --name mmdvmhost \
  --privileged \
  --device=/dev/ttyUSB0:/dev/ttyUSB0 \
  ghcr.io/mw0mwz/mmdvmhost:latest
  