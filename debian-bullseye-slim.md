### Dockerfile

```dockerfile
# Use the official Debian "bullseye-slim" image for a minimal footprint.
FROM debian:bullseye-slim

# Accept build arguments for user and group IDs, defaulting to 1000.
ARG UID=1000
ARG GID=1000

# Set environment variables to prevent interactive prompts during installation.
ENV DEBIAN_FRONTEND=noninteractive

# Update package lists, install necessary tools, and clean up in a single layer.
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3 \
    python3-pip \
    git \
    curl \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Create a group and user with the specified UID/GID from the build arguments.
RUN groupadd -g $GID devgroup && \
    useradd -u $UID -g $GID -ms /bin/bash devuser

# Create the application directory and set its ownership.
RUN mkdir -p /app && chown -R devuser:devgroup /app

# Switch to the non-root user.
USER devuser

# Set the working directory inside the container.
WORKDIR /app

# Set the default command to open a bash shell.
CMD ["bash"]
```

### docker-compose.yml

This `docker-compose.yml` file is updated to align with modern best practices.

```yaml
services:
  debian-dev:
    build:
      context: .
      # Pass the host user's UID and GID as build arguments.
      # This uses shell variable substitution, falling back to 1000 if not set.
      args:
        UID: ${UID:-1000}
        GID: ${GID:-1000}
    # Keep STDIN open for an interactive shell.
    stdin_open: true
    # Allocate a pseudo-TTY.
    tty: true
    # Mount the local './src' directory into the container's '/app' directory.
    volumes:
      - ./src:/app
```

### Usage Instructions

The usage instructions remain the same and provide a complete workflow.

1.  **Prepare Your Environment (One-time setup):**
    To ensure correct file permissions between your host machine and the container, run the following command in your terminal. It creates a `.env` file that Docker Compose will automatically use to get your user and group IDs.
    ```sh
    echo -e "UID=$(id -u)\nGID=$(id -g)" > .env
    ```

2.  **Directory Structure:**
    Your project should now have the following structure:
    ```
    .
    ├── .env
    ├── docker-compose.yml
    ├── Dockerfile
    └── src/
    ```

3.  **Build and Start the Container:**
    Open a terminal in the project's root directory and run the following command. This will build the image with your user permissions and start the container in detached mode.
    ```sh
    docker-compose up -d --build
    ```

4.  **Access the Interactive Shell:**
    To get an interactive `bash` shell inside the running container, execute this command:
    ```sh
    docker-compose exec debian-dev bash
    ```
    You can now create and modify files in the `/app` directory (and your local `./src` directory) without any permission errors on either the host or in the container.
