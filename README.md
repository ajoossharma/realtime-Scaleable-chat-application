# Realtime Scalable Chat Application

A highly scalable real-time chat application built with WebSockets, Kafka, and Next.js in a monorepo architecture.

![Project Architecture](public/circles.svg)

## Overview

This project demonstrates how to build a scalable WebSocket-based chat system that can handle thousands of concurrent connections. It uses Kafka as a message broker to distribute messages across multiple server instances, allowing for horizontal scaling.

## Features

- **Real-time Communication**: Instant messaging using WebSockets
- **Horizontal Scalability**: Multiple server instances with Kafka message distribution
- **Persistence**: Chat history stored in a PostgreSQL database via Prisma ORM
- **Modern UI**: Clean, responsive interface built with Next.js and custom UI components
- **Monorepo Structure**: Organized with Turborepo for efficient development workflow

## Tech Stack

### Frontend
- Next.js (React framework)
- TypeScript
- Custom UI component library
- WebSocket client (via custom SocketProvider context)

### Backend
- Node.js with TypeScript
- WebSocket server implementation
- Kafka for message distribution
- Prisma ORM
- PostgreSQL database

### DevOps & Tooling
- Turborepo for monorepo management
- ESLint for code linting
- Shared TypeScript configurations
- Custom generators for component creation

## Project Structure

```
📦 Realtime-scalable-chat-app
 ┣ 📂 apps
 ┃ ┣ 📂 docs            # Documentation site
 ┃ ┣ 📂 server          # WebSocket server and Kafka integration
 ┃ ┗ 📂 web             # Next.js client application
 ┣ 📂 packages
 ┃ ┣ 📂 eslint-config   # Shared ESLint configurations
 ┃ ┣ 📂 typescript-config # Shared TypeScript configurations
 ┃ ┗ 📂 ui              # Shared UI component library
 ┣ 📜 turbo.json        # Turborepo configuration
 ┗ 📜 package.json      # Root package.json
```

## Getting Started

### Prerequisites

- Node.js 16.x or later
- Yarn package manager
- Docker (for running Kafka and PostgreSQL)
- PostgreSQL database

### Installation

1. Clone the repository
   ```bash
   git clone https://github.com/yourusername/realtime-scalable-chat-app.git
   cd realtime-scalable-chat-app
   ```

2. Install dependencies
   ```bash
   yarn install
   ```

3. Set up environment variables
   
   Create a `.env` file in the `apps/server` directory:
   ```
   DATABASE_URL="postgresql://postgres:password@localhost:5432/chat"
   KAFKA_BROKERS="localhost:9092"
   PORT=4000
   ```

   Create a `.env.local` file in the `apps/web` directory:
   ```
   NEXT_PUBLIC_WEBSOCKET_URL="ws://localhost:4000"
   ```

4. Start the database and Kafka (using Docker)
   ```bash
   docker-compose up -d
   ```

5. Run database migrations
   ```bash
   cd apps/server
   npx prisma migrate dev
   ```

6. Start the development servers
   ```bash
   # From the root directory
   yarn dev
   ```

## System Design

### High-Level Architecture

```mermaid
graph TB
    subgraph "Client Layer"
        C1[Client 1<br/>Next.js App]
        C2[Client 2<br/>Next.js App]
        C3[Client N<br/>Next.js App]
    end
    
    subgraph "Load Balancer"
        LB[Load Balancer<br/>Nginx/HAProxy]
    end
    
    subgraph "WebSocket Server Layer"
        WS1[WebSocket Server 1<br/>Node.js + Socket.IO]
        WS2[WebSocket Server 2<br/>Node.js + Socket.IO]
        WS3[WebSocket Server N<br/>Node.js + Socket.IO]
    end
    
    subgraph "Message Broker"
        K1[Kafka Broker 1]
        K2[Kafka Broker 2]
        K3[Kafka Broker N]
        KT[Chat Messages Topic]
    end
    
    subgraph "Database Layer"
        DB[(PostgreSQL<br/>Chat History)]
        PRISMA[Prisma ORM]
    end
    
    subgraph "Caching Layer"
        REDIS[(Redis<br/>Session & Active Users)]
    end
    
    C1 -.->|WebSocket| LB
    C2 -.->|WebSocket| LB
    C3 -.->|WebSocket| LB
    
    LB --> WS1
    LB --> WS2
    LB --> WS3
    
    WS1 <--> K1
    WS2 <--> K2
    WS3 <--> K3
    
    K1 --- KT
    K2 --- KT
    K3 --- KT
    
    WS1 --> PRISMA
    WS2 --> PRISMA
    WS3 --> PRISMA
    
    PRISMA --> DB
    
    WS1 -.-> REDIS
    WS2 -.-> REDIS
    WS3 -.-> REDIS
    
    style C1 fill:#e1f5fe
    style C2 fill:#e1f5fe
    style C3 fill:#e1f5fe
    style WS1 fill:#f3e5f5
    style WS2 fill:#f3e5f5
    style WS3 fill:#f3e5f5
    style K1 fill:#fff3e0
    style K2 fill:#fff3e0
    style K3 fill:#fff3e0
    style DB fill:#e8f5e8
    style REDIS fill:#ffebee
```

### Message Flow Architecture

```mermaid
sequenceDiagram
    participant C1 as Client 1
    participant WS1 as WebSocket Server 1
    participant K as Kafka
    participant DB as PostgreSQL
    participant WS2 as WebSocket Server 2
    participant C2 as Client 2
    
    Note over C1,C2: User sends a message
    
    C1->>WS1: Send Message via WebSocket
    
    Note over WS1: Message Processing
    WS1->>WS1: Validate & Process Message
    
    par Persist to Database
        WS1->>DB: Store Message (via Prisma)
        DB-->>WS1: Confirm Storage
    and Publish to Kafka
        WS1->>K: Produce Message to Topic
        K-->>WS1: Confirm Publish
    end
    
    Note over K: Message Distribution
    K->>WS1: Consume Message
    K->>WS2: Consume Message
    
    Note over WS1,WS2: Broadcast to Connected Clients
    WS1->>C1: Broadcast Message
    WS2->>C2: Broadcast Message
    
    Note over C1,C2: Real-time message delivery
```

### Component Architecture

```mermaid
graph LR
    subgraph "Frontend (Next.js)"
        SC[SocketProvider Context]
        UI[Chat UI Components]
        SC --> UI
    end
    
    subgraph "Backend Services"
        WS[WebSocket Handler]
        KP[Kafka Producer]
        KC[Kafka Consumer]
        PS[Prisma Service]
        
        WS --> KP
        KC --> WS
        WS --> PS
    end
    
    subgraph "External Services"
        KF[Kafka Cluster]
        PG[(PostgreSQL)]
    end
    
    UI -.->|WebSocket Connection| WS
    KP --> KF
    KF --> KC
    PS --> PG
    
    style SC fill:#e3f2fd
    style UI fill:#e3f2fd
    style WS fill:#f1f8e9
    style KP fill:#fff8e1
    style KC fill:#fff8e1
    style PS fill:#fce4ec
```

## Architecture

### WebSocket Server

The server application uses a WebSocket implementation to handle real-time communication with clients. When a message is received from a client:

1. The message is validated and processed
2. The message is produced to a Kafka topic
3. The message is saved to the PostgreSQL database using Prisma

The server also consumes messages from Kafka topics and broadcasts them to connected clients.

### Kafka Integration

Kafka serves as the message broker between multiple server instances:

- Each server instance produces messages to shared Kafka topics
- Each server instance consumes messages from these topics
- This allows messages to be properly distributed among all server instances

### Client Application

The Next.js client application:

- Connects to the WebSocket server using a custom `SocketProvider` context
- Handles real-time message updates
- Provides a user interface for sending and receiving messages

## Scaling Considerations

This architecture allows for horizontal scaling by:

- Adding more WebSocket server instances behind a load balancer
- Using Kafka to ensure messages are distributed to all connected clients
- Maintaining shared state across server instances

## Development Workflow

This project uses Turborepo to manage the monorepo structure:

```bash
# Run all applications in development mode
yarn dev

# Build all applications
yarn build

# Run linting across all projects
yarn lint

# Generate a new UI component
cd packages/ui
yarn turbo gen

# Add a new dependency to a workspace
yarn workspace @repo/web add package-name
```

## Deployment

### Server Deployment

The WebSocket server can be deployed to any Node.js hosting platform:

1. Build the server
   ```bash
   cd apps/server
   yarn build
   ```

2. Start the server
   ```bash
   yarn start
   ```

### Web Application Deployment

The Next.js web application can be deployed to Vercel:

```bash
cd apps/web
vercel
```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgments

- [Turborepo](https://turbo.build/) for the monorepo structure
- [Next.js](https://nextjs.org/) for the frontend framework
- [Prisma](https://www.prisma.io/) for database ORM
- [Kafka](https://kafka.apache.org/) for message distribution
