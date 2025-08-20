# Build stage
FROM --platform=$BUILDPLATFORM alpine:latest AS builder

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
# These are NOT published to host unless explicitly mapped
EXPOSE 62030/udp
EXPOSE 42000/udp  
EXPOSE 20010/udp

CMD ["/usr/local/bin/MMDVMHost", "/etc/mmdvm/MMDVM.ini"]
