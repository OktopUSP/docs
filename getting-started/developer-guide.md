# Developer Guide

### Project Architecture

#### High-Level Structure

```
oktopus/
├── agent/          # USP Agent configuration and examples
├── backend/        # Core backend services
│   └── services/
│       ├── controller/    # Main controller service
│       └── mtp/          # Message Transfer Protocol adapters
├── frontend/       # Next.js web application
├── deploy/         # Deployment configurations
└── docs/           # GitBook documentation
```

#### Backend Architecture

The backend is built with Go and follows a modular service architecture:

**Controller Service**

Located in `backend/services/controller/`, this is the core service that manages devices and handles protocol communications.

**Key Components:**

* **API Layer** (`internal/api/`): REST API endpoints for device management
  * `api.go`: Main API router and middleware setup
  * Device message handlers (GET, SET, ADD, DELETE operations)
* **CWMP Support** (`internal/cwmp/`): TR-069 protocol implementation
  * Handles CWMP-specific operations like GetParameterNames, SetParameterValues
* **Entity Layer** (`internal/entity/`): Core business entities and data models
* **Database Layer** (`internal/db/`): Database abstraction and persistence
* **Bridge** (`internal/bridge/`): Communication bridge between services

**MTP Adapters**

The Message Transfer Protocol adapters handle different communication protocols:

* **WebSocket Adapter** (`backend/services/mtp/ws-adapter/`): Implements WebSocket MTP for USP
* **MQTT Adapter**: Handles MQTT-based communication
* **STOMP Adapter**: Handles STOMP protocol

As mentioned in the USP MTP documentation, USP supports multiple MTPs including WebSockets, MQTT, and STOMP.

#### Frontend Architecture

The frontend is built with Next.js and located in `frontend/`. It provides a web interface for managing devices.

### Development Setup

#### Prerequisites

* **Go**: Version 1.23+
* **Node.js**: Version 18+
* **NATS**: Message broker for inter-service communication
* **Docker**: For containerized development (optional)

#### Building the Services

**Docker Development**

From the root `build/` directory, you can build all services at once:

```bash
cd build/
make build              # Build all services (backend + frontend)
make build-backend      # Build only backend services
make build-frontend     # Build only frontend
```

**Building Individual Microservices**

Each microservice has its own build directory with a Makefile and Dockerfile. The general pattern is:

```bash
cd backend/services/<service-name>/build/
make build
```

**Available Backend Services:**

1.  **Controller Service** - Core device management

    ```bash
    cd backend/services/controller/build/
    make build
    ```

    * Built from: `cmd/controller/main.go`
    * Binary: `controller`
    * Base image: Alpine Linux
2.  **ACS Service** - Auto Configuration Server (CWMP)

    ```bash
    cd backend/services/acs/build/
    make build
    ```

    * Built from: `cmd/acs/main.go`
    * Binary: `acs`
    * Base image: Alpine Linux
3.  **MTP Adapters** - Message Transfer Protocol handlers

    **WebSocket Adapter:**

    ```bash
    cd backend/services/mtp/ws-adapter/build/
    make build
    ```

    * Built from: `cmd/ws-adapter/main.go`
    * Handles WebSocket connections for USP

    **WebSocket Service:**

    ```bash
    cd backend/services/mtp/ws/build/
    make build
    ```

    * Built from: `cmd/ws/main.go`

    **MQTT Adapter:**

    ```bash
    cd backend/services/mtp/mqtt-adapter/build/
    make build
    ```

    * Built from: `cmd/mqtt-adapter/main.go`
    * Handles MQTT connections for USP

    **MQTT Service:**

    ```bash
    cd backend/services/mtp/mqtt/build/
    make build
    ```

    * Built from: `cmd/mqtt/main.go`

    **STOMP Adapter:**

    ```bash
    cd backend/services/mtp/stomp-adapter/build/
    make build
    ```

    * Built from: `cmd/stomp-adapter/main.go`
    * Handles STOMP connections for USP

    **STOMP Service:**

    ```bash
    cd backend/services/mtp/stomp/build/
    make build
    ```

    * Built from: `cmd/stomp/main.go`

    **Generic Adapter:**

    ```bash
    cd backend/services/mtp/adapter/build/
    make build
    ```

    * Built from: `cmd/adapter/main.go`
4.  **Utility Services**

    **Socket.IO Service:**

    ```bash
    cd backend/services/utils/socketio/build/
    make build
    ```

    * Node.js service for real-time communication
    * Base image: Node 16.20.2-alpine

    **File Server:**

    ```bash
    cd backend/services/utils/file-server/build/
    make build
    ```

    * Serves firmware files and other static content
5.  **Bulk Data Collector**

    ```bash
    cd backend/services/bulkdata/http/build/
    make build
    ```

    * Built from: `cmd/http-bulk-collector/main.go`
    * Collects bulk data from devices

**Building the Frontend**

```bash
cd frontend/build/
make build
```

* Next.js application
* Node.js 18.18.0-alpine base image
* Production build with optimizations

### **Local Development**

**Backend Services (Go):**

```bash
# Navigate to the service directory
cd backend/services/controller  # or any other service

# Install dependencies
go mod download

# Run locally
go run cmd/controller/main.go  # adjust path for the specific service cmd/{service}/main.go

# Build binary
go build -o controller cmd/controller/main.go
# Run binary
./controller

# Run without build (for development purposes)
go run cmd/controller/main.go
```

**Frontend (Next.js):**

```bash
cd frontend

# Install dependencies
npm install

# Development server
npm run dev

# Production build
npm run build
npm run start
```

**Makefile Commands**

Each service's Makefile supports these commands:

* `make build` - Build Docker image
* `make push` - Push image to registry
* `make run` - Create and start container
* `make stop` - Stop running container
* `make remove` - Remove container
* `make delete` - Delete Docker image
* `make logs` - Show container logs
* `make bash` - Access container shell
* `make release` - Tag as latest and push

**Customization:**

```bash
# Build with custom Docker user
make build DOCKER_USER=myuser

# Build with specific tag
make build DOCKER_TAG=v1.0.0

# Use different shell for container access
make bash CONTAINER_SHELL=/bin/bash
```

#### Running the Services

The project includes:

* Build configurations in `build/Makefile`
* Docker Compose deployment in `deploy/compose/`
* Kubernetes deployment in `deploy/kubernetes/`

### Code Organization

#### Backend Service Structure

The controller service follows a clean architecture pattern:

```
backend/services/controller/
├── cmd/                    # Application entry points
├── internal/
│   ├── api/               # HTTP API handlers
│   ├── bridge/            # Service communication bridge
│   ├── config/            # Configuration management
│   ├── cwmp/              # CWMP protocol implementation
│   ├── db/                # Database layer
│   ├── entity/            # Domain entities
│   └── utils/             # Utility functions
└── main.go
```

#### Pull Request Process

1. Fork the repository and create a feature branch
2. Make your changes following the code style guidelines
3. Write clear commit messages describing what and why
4. Test thoroughly with both protocols if applicable
5. Update documentation if you're adding new features
6. Submit a pull request to the main repository

Include in your PR description:

* What problem does it solve?
* How was it tested?
* Any breaking changes?
* Screenshots (for UI changes)

### Understanding Key Concepts

#### Device Communication Flow

1. Device connects via MTP adapter (WebSocket/MQTT/STOMP for USP, or ACS connection for CWMP)
2. Adapter ingest and digest messages through NATS message broker
3. API endpoint at Controller processes the user requests
4. Message is queued and sent to the device
5. Response is awaited with timeout
6. Result is returned to the client

#### CWMP vs USP Implementation

The codebase supports both protocols:

**CWMP (TR-069)**:

* Synchronous request/response model
* XML-based RPC
* Direct parameter paths (e.g., `InternetGatewayDevice.LANDevice.*.WLANConfiguration.*`)
* See `internal/cwmp/` for implementation

**USP (TR-369)**:

* Supports multiple MTPs (MQTT, WebSocket, STOMP)
* Protocol Buffers encoding
* More flexible messaging model

### Documentation

When adding features, update:

1. **Code comments**: Especially for exported functions
2. **GitBook docs** in `docs/`: Add user-facing documentation
3. **README files**: Update relevant README files
4. **API documentation**: If adding new endpoints

### Resources

* **Main Repository**: [github.com/OktopUSP/oktopus](https://github.com/OktopUSP/oktopus)
* **Documentation**: [github.com/OktopUSP/docs](https://github.com/OktopUSP/docs)
* **Community Slack**: Join the [Oktopus Slack](https://join.slack.com/t/oktopustr-369/shared_invite/zt-1znmrbr52-3AXgOlSeQTPQW8_Qhn3C4g)
* **Rest API Spec**: [Oktopus Postman Documentation](https://documenter.getpostman.com/view/18932104/2s93eR3vQY)
* **USP Specification**: Broadband Forum TR-369
* **CWMP Specification**: Broadband Forum TR-069

### Getting Help

* **Community Support**: Available on [Slack](https://join.slack.com/t/oktopustr-369/shared_invite/zt-1znmrbr52-3AXgOlSeQTPQW8_Qhn3C4g)
* **Enterprise Support**: [support@oktopus.app.br](mailto:support@oktopus.app.br)
* **Issues**: Report bugs on [GitHub Issues](https://github.com/OktopUSP/oktopus/issues)

### License

Oktopus is released under the license specified in the `LICENSE` file. Make sure your contributions comply with this license.
