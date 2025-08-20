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

# Clone and build P25Clients (contains P25Gateway)
WORKDIR /build
ARG CACHEBUST=1
RUN echo "Cache bust: $CACHEBUST"
RUN git clone --depth=1 https://github.com/g4klx/P25Clients.git
WORKDIR /build/P25Clients/P25Gateway
RUN make clean && make

# Strip the binary to reduce size
RUN strip P25Gateway

# Runtime stage  
FROM alpine:latest

# Install runtime dependencies only
RUN apk add --no-cache \
    libstdc++ \
    libgcc \
    tzdata

# Create Pi-Star directory structure including P25 Audio
RUN mkdir -p /opt/pi-star/bin \
             /opt/pi-star/etc \
             /opt/pi-star/log \
             /opt/pi-star/P25Audio

# Create non-root user
RUN addgroup -g 1001 pistar && \
    adduser -D -u 1001 -G pistar pistar

# Copy binary from build stage to Pi-Star bin directory
COPY --from=builder /build/P25Clients/P25Gateway/P25Gateway /opt/pi-star/bin/

# Copy audio files from build stage to P25Audio directory
COPY --from=builder /build/P25Clients/P25Gateway/Audio/* /opt/pi-star/P25Audio/

# Set ownership of Pi-Star directories
RUN chown -R pistar:pistar /opt/pi-star

# Add Pi-Star bin to PATH
ENV PATH="/opt/pi-star/bin:$PATH"

# Switch to non-root user
USER pistar

# Set working directory
WORKDIR /opt/pi-star

# Expose ports for inter-container communication
# P25 gateway communication
EXPOSE 42020/udp
# Remote Control
EXPOSE 6074/udp

# Use the Pi-Star path structure
CMD ["/opt/pi-star/bin/P25Gateway", "/opt/pi-star/etc/P25Gateway.ini"]