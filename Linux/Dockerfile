FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies and useful tools
RUN apt update && apt install -y \
    libglib2.0-0 \
    libgtk-3-0 \
    libxkbcommon0 \
    libnss3 \
    libx11-6 \
    libxcomposite1 \
    libxdamage1 \
    libxrandr2 \
    libasound2 \
    libxext6 \
    libxtst6 \
    libatk1.0-0 \
    libatk-bridge2.0-0 \
    libcups2 \
    libgbm1 \
    wget \
    ca-certificates \
    fuse \
    xdg-utils \
    libwayland-client0 \
    libwayland-cursor0 \
    libwayland-egl1 \
    libwayland-server0 \
    libdrm2 \
    mesa-utils \
    dbus-x11 \
    libdbus-1-3 \
    xvfb \
    x11-xserver-utils \
    pulseaudio \
    locales \
    software-properties-common \
    gnupg \
    curl \
    sudo \
    vim \
    nano \
    git \
    htop \
    zip \
    unzip \
    python3 \
    python3-pip \
 && rm -rf /var/lib/apt/lists/*

# Install Firefox directly from Mozilla PPA instead of snap
RUN add-apt-repository ppa:mozillateam/ppa -y && \
    echo 'Package: *\nPin: release o=LP-PPA-mozillateam\nPin-Priority: 1001' > /etc/apt/preferences.d/mozilla-firefox && \
    apt update && apt install -y firefox && \
    rm -rf /var/lib/apt/lists/*

# Set up locale
RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Create a non-root user with the same UID as your host user
RUN useradd -u 1000 -ms /bin/bash cursoruser

# Add the user to the sudo group and configure passwordless sudo
RUN usermod -aG sudo cursoruser && \
    echo "cursoruser ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/cursoruser && \
    chmod 0440 /etc/sudoers.d/cursoruser

# Create necessary directories and set permissions
RUN mkdir -p /home/cursoruser/.cursor && \
    mkdir -p /home/cursoruser/.cursor/extensions && \
    mkdir -p /home/cursoruser/.config/Cursor && \
    mkdir -p /home/cursoruser/.config/Cursor/Code\ Cache/js && \
    mkdir -p /home/cursoruser/.config/Cursor/Code\ Cache/wasm && \
    mkdir -p /home/cursoruser/.config/Cursor/User/globalStorage && \
    mkdir -p /home/cursoruser/.mozilla/firefox && \
    chown -R cursoruser:cursoruser /home/cursoruser/.config && \
    chown -R cursoruser:cursoruser /home/cursoruser/.cursor && \
    chown -R cursoruser:cursoruser /home/cursoruser/.mozilla && \
    chmod -R 755 /home/cursoruser/.cursor && \
    chmod -R 755 /home/cursoruser/.config/Cursor

# Set work directory for that user
USER cursoruser
WORKDIR /home/cursoruser

# Setup custom prompt to show cursor-container instead of host name
RUN echo 'export PS1="\[\033[01;32m\]\u@cursor-container\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ "' >> /home/cursoruser/.bashrc

# Copy Cursor.AppImage with correct permissions
COPY --chown=cursoruser:cursoruser Cursor.AppImage ./Cursor.AppImage
RUN chmod +x Cursor.AppImage

# Create optimized startup script
RUN echo '#!/bin/bash\n\
# Ensure permissions are correct at startup\n\
sudo chown -R cursoruser:cursoruser /home/cursoruser/.cursor\n\
sudo chown -R cursoruser:cursoruser /home/cursoruser/.config/Cursor\n\
sudo chmod -R 755 /home/cursoruser/.cursor\n\
sudo chmod -R 755 /home/cursoruser/.config/Cursor\n\
\n\
# Function to start Firefox when needed\n\
start_firefox() {\n\
  echo "Starting Firefox for authentication..."\n\
  /usr/bin/firefox --new-instance --private-window & \n\
  return $!\n\
}\n\
\n\
# Start Cursor\n\
echo "Starting Cursor..."\n\
./Cursor.AppImage --no-sandbox\n\
' > /home/cursoruser/start-cursor.sh && chmod +x /home/cursoruser/start-cursor.sh

# Setup environment variables for browser handling
ENV BROWSER="/usr/bin/firefox"
ENV ELECTRON_DEFAULT_ERROR_MODE=1
ENV ELECTRON_ENABLE_LOGGING=1

# Default command - use our wrapper script
CMD ["/home/cursoruser/start-cursor.sh"]