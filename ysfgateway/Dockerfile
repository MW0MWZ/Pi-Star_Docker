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

# Clone and build YSFClients (contains YSFGateway)
WORKDIR /build
RUN git clone https://github.com/g4klx/YSFClients.git
WORKDIR /build/YSFClients/YSFGateway
RUN make clean && make

# Strip the binary to reduce size
RUN strip YSFGateway

# Runtime stage  
FROM alpine:latest

# Install runtime dependencies only
RUN apk add --no-cache \
    libstdc++ \
    libgcc \
    tzdata

# Create Pi-Star directory structure including YSF Audio
RUN mkdir -p /opt/pi-star/bin \
             /opt/pi-star/etc \
             /opt/pi-star/log

# Create non-root user
RUN addgroup -g 1001 pistar && \
    adduser -D -u 1001 -G pistar pistar

# Copy binary from build stage to Pi-Star bin directory
COPY --from=builder /build/YSFClients/YSFGateway/YSFGateway /opt/pi-star/bin/

# Set ownership of Pi-Star directories
RUN chown -R pistar:pistar /opt/pi-star

# Add Pi-Star bin to PATH
ENV PATH="/opt/pi-star/bin:$PATH"

# Switch to non-root user
USER pistar

# Set working directory
WORKDIR /opt/pi-star

# Expose ports for inter-container communication
# YSF gateway communication
EXPOSE 4200/udp
# Remote control
EXPOSE 6073/udp

# Use the Pi-Star path structure
CMD ["/opt/pi-star/bin/YSFGateway", "/opt/pi-star/etc/YSFGateway.ini"]