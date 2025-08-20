# Build stage
FROM alpine:latest AS builder

# Install build dependencies including wxWidgets
RUN apk add --no-cache \
    git \
    gcc \
    g++ \
    make \
    cmake \
    linux-headers \
    musl-dev \
    binutils \
    wxwidgets-dev

# Build arguments for cross-compilation
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "Building for $TARGETPLATFORM on $BUILDPLATFORM"

# Clone and build ircDDBGateway
WORKDIR /build
RUN git clone https://github.com/g4klx/ircDDBGateway.git
WORKDIR /build/ircDDBGateway

# Create Pi-Star directory structure for build
RUN mkdir -p /opt/pi-star/bin \
             /opt/pi-star/etc/ircDDBGateway \
             /opt/pi-star/log

# Fix the Config
RUN sed -i 's/^export DATADIR.*/export DATADIR \?= \/opt\/pi-star\/etc\/ircDDBGateway/g' Makefile 
RUN sed -i 's/^export LOGDIR.*/export LOGDIR  \?= \/opt\/pi-star\/log/g' Makefile
RUN sed -i 's/^export CONFDIR.*/export CONFDIR \?= \/opt\/pi-star\/etc/g' Makefile
RUN sed -i 's/^export BINDIR.*/export BINDIR  \?= \/opt\/pi-star\/bin/g' Makefile

# Build the project
RUN make clean && make -j $(nproc) && make install

# Strip the binaries to reduce size
RUN find /opt/pi-star/bin -type f -executable -exec strip {} \;

# Runtime stage  
FROM alpine:latest

# Install runtime dependencies including wxWidgets runtime
RUN apk add --no-cache \
    libstdc++ \
    libgcc \
    tzdata \
    wxwidgets

# Create Pi-Star directory structure
RUN mkdir -p /opt/pi-star/bin \
             /opt/pi-star/etc/ircDDBGateway \
             /opt/pi-star/log

# Create non-root user
RUN addgroup -g 1001 pistar && \
    adduser -D -u 1001 -G pistar pistar

# Copy all Pi-Star files from build stage
COPY --from=builder /opt/pi-star/ /opt/pi-star/

# Set ownership of Pi-Star directories
RUN chown -R pistar:pistar /opt/pi-star

# Add Pi-Star bin to PATH
ENV PATH="/opt/pi-star/bin:$PATH"

# Switch to non-root user
USER pistar

# Set working directory
WORKDIR /opt/pi-star

# Expose ports for inter-container communication
# ircDDB communication ports
EXPOSE 9007/tcp
EXPOSE 20001/udp

# Use the Pi-Star path structure - adjust binary name if needed
CMD ["/opt/pi-star/bin/ircDDBGatewayd", "-v"]