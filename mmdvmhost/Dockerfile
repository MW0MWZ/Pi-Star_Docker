# Build stage - use target platform, not build platform
FROM alpine:latest AS builder

# Install build dependencies
RUN apk add --no-cache \
    git \
    gcc \
    g++ \
    make \
    cmake \
    linux-headers \
    musl-dev \
    binutils

# Build arguments for cross-compilation
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "Building for $TARGETPLATFORM on $BUILDPLATFORM"

# Clone and build MMDVMHost
WORKDIR /build
RUN git clone https://github.com/g4klx/MMDVMHost.git
WORKDIR /build/MMDVMHost
RUN make clean && make

# Strip the binary to reduce size
RUN strip MMDVMHost

# Runtime stage  
FROM alpine:latest

# Install runtime dependencies only
RUN apk add --no-cache \
    libstdc++ \
    libgcc \
    tzdata

# Create non-root user
RUN addgroup -g 1001 mmdvm && \
    adduser -D -u 1001 -G mmdvm mmdvm

# Copy stripped binary from build stage
COPY --from=builder /build/MMDVMHost/MMDVMHost /usr/local/bin/

# Create directories for config and logs
RUN mkdir -p /etc/mmdvm /var/log/mmdvm && \
    chown -R mmdvm:mmdvm /etc/mmdvm /var/log/mmdvm

# Switch to non-root user
USER mmdvm

# Expose ports for inter-container communication
# Modem
EXPOSE 3335/udp
# Transparent Data
EXPOSE 40095/udp
# D-Star
EXPOSE 20011/udp
# DMR
EXPOSE 62032/udp
# Fusion
EXPOSE 3200/udp
# P25
EXPOSE 32010/udp
# NXDN
EXPOSE 14021/udp
# POCSAG
EXPOSE 3800/udp
# FM
EXPOSE 3810/udp
# Remote Control
EXPOSE 7642/udp

# Entry Point
CMD ["/usr/local/bin/MMDVMHost"]
