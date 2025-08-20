# Build stage
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
ARG CACHEBUST=1
RUN echo "Cache bust: $CACHEBUST"
RUN git clone --depth=1 https://github.com/g4klx/MMDVMHost.git
RUN git clone --depth=1 https://github.com/g4klx/MMDVMCal.git

# Build MMDVMHost
WORKDIR /build/MMDVMHost
RUN make clean && make
RUN strip MMDVMHost RemoteCommand

# Build MMDVMCal
WORKDIR /build/MMDVMCal
RUN make clean && make
RUN strip MMDVMCal

# Runtime stage  
FROM alpine:latest

# Install runtime dependencies only
RUN apk add --no-cache \
    libstdc++ \
    libgcc \
    tzdata

# Create Pi-Star directory structure
RUN mkdir -p /opt/pi-star/bin \
             /opt/pi-star/etc \
             /opt/pi-star/log

# Create non-root user
RUN addgroup -g 1001 pistar && \
    adduser -D -u 1001 -G pistar pistar

# Copy binaries from build stage to Pi-Star bin directory
COPY --from=builder /build/MMDVMHost/MMDVMHost /opt/pi-star/bin/
COPY --from=builder /build/MMDVMHost/RemoteCommand /opt/pi-star/bin/
COPY --from=builder /build/MMDVMCal/MMDVMCal /opt/pi-star/bin/

# Set ownership of Pi-Star directories
RUN chown -R pistar:pistar /opt/pi-star

# Add Pi-Star bin to PATH
ENV PATH="/opt/pi-star/bin:$PATH"

# Switch to non-root user
USER pistar

# Set working directory
WORKDIR /opt/pi-star

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

# Entrypoint
CMD ["/opt/pi-star/bin/MMDVMHost", "/opt/pi-star/etc/MMDVM.ini"]
