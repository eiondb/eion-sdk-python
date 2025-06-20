# Eion Python SDK

Python SDK for Eion - Shared memory storage and collaborative intelligence for AI agent systems.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Authentication](#authentication)
- [Core Concepts](#core-concepts)
- [API Reference](#api-reference)
- [Troubleshooting](#troubleshooting)
- [Configuration](#configuration)
- [Features](#features)
- [License](#license)

## Prerequisites

Before using this SDK, you need to have an **Eion server running**. This SDK is a client that connects to your Eion server instance.

### Docker

```bash
# 1. Create a docker-compose.yml file
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  eion-server:
    image: eiondb/eion:latest
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://eion:password@postgres:5432/eion
      - CLUSTER_API_KEY=my-secret-api-key-123  # You choose this!
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=eion
      - POSTGRES_USER=eion
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
EOF

# 2. Start the Eion server
docker-compose up -d

# 3. Verify it's running
curl http://localhost:8080/health
```

## Installation

```bash
pip install eiondb
```

Or install from source:
```bash
git clone https://github.com/eiondb/eion-sdk-python.git
cd eion-sdk-python
pip install -e .
```

## Quick Start

**Important**: Make sure your Eion server is running first (see Prerequisites above).

```python
from eiondb import EionClient

# Initialize client - use the same API key you set in your server config
client = EionClient(
    base_url="http://localhost:8080",
    cluster_api_key="my-secret-api-key-123"
)

# Check server health (this will fail if the API key doesn't match)
health = client.health_check()
print(f"Server status: {health['status']}")

# Create a user
user = client.create_user(
    user_id="demo_user",
    name="Demo User"
)

# Register an agent
agent = client.register_agent(
    agent_id="demo_agent",
    name="Demo Agent",
    permission="crud",
    description="A demo agent for testing"
)

# Create a session
session = client.create_session(
    session_id="demo_session",
    user_id="demo_user",
    session_name="Demo Session"
)
```

## Authentication

### Understanding the Cluster API Key

The `cluster_api_key` is a **secret key that you define** when setting up your Eion server. It's not provided by a service - you choose it yourself for security.

- **In your server**: Set `CLUSTER_API_KEY=my-secret-api-key-123` 
- **In your SDK**: Use the same `cluster_api_key="my-secret-api-key-123"`

### Authentication Methods

The SDK supports multiple ways to provide your credentials:

### 1. Direct Parameters
```python
client = EionClient(
    base_url="http://localhost:8080",
    cluster_api_key="my-secret-api-key-123"
)
```

### 2. Environment Variables
```bash
export EION_BASE_URL="http://localhost:8080"
export EION_CLUSTER_API_KEY="my-secret-api-key-123"
```

```python
client = EionClient()  # Auto-loads from environment
```

### 3. Configuration File
Create `eion.yaml`:
```yaml
server:
  base_url: "http://localhost:8080"
auth:
  cluster_api_key: "my-secret-api-key-123"
```

```python
client = EionClient(config_file="eion.yaml")
```

## Core Concepts

### Users
Users represent end-users of your application who will interact with agents.

### Agents
Agents are AI entities that can participate in sessions and perform operations based on their permissions.

### Sessions
Sessions are conversation contexts where users and agents interact.

### Agent Groups
Groups of agents that can be managed together and assigned to session types.

### Session Types
Templates that define which agent groups can participate in sessions.

## API Reference

### Health Check

```python
# Check server health
health = client.health_check()
# Returns: {"status": "healthy", "timestamp": "..."}
```

### User Management

```python
# Create user
user = client.create_user(
    user_id="user123",
    name="John Doe"  # Optional
)

# Delete user
client.delete_user("user123")
```

### Agent Management

```python
# Register agent
agent = client.register_agent(
    agent_id="agent123",
    name="My Agent",
    permission="crud",  # "r", "cr", "crud"
    description="Agent description",  # Optional
    guest=False  # Optional, default False
)

# Get agent details
agent = client.get_agent("agent123")

# Update agent
updated_agent = client.update_agent(
    "agent123",
    name="Updated Name",
    description="New description"
)

# List agents
agents = client.list_agents()
# With filters:
agents = client.list_agents(permission="crud", guest=False)

# Delete agent
client.delete_agent("agent123")
```

### Session Management

```python
# Create session
session = client.create_session(
    session_id="session123",
    user_id="user123",
    session_type_id="default",  # Optional
    session_name="My Session"   # Optional
)

# Delete session
client.delete_session("session123")
```

### Agent Group Management

```python
# Register agent group
group = client.register_agent_group(
    agent_group_id="group123",
    name="Customer Support Team",
    agent_ids=["agent1", "agent2"],  # Optional
    description="Support agents"     # Optional
)

# List agent groups
groups = client.list_agent_groups()

# Get group details
group = client.get_agent_group("group123")

# Update agent group
updated_group = client.update_agent_group(
    "group123",
    name="Updated Team Name",
    agent_ids=["agent1", "agent2", "agent3"]
)

# Delete agent group
client.delete_agent_group("group123")
```

### Session Type Management

```python
# Register session type
session_type = client.register_session_type(
    session_type_id="support_session",
    name="Customer Support",
    agent_group_ids=["support_team"],  # Optional
    description="Customer support sessions",  # Optional
    encryption="SHA256"  # Optional, default SHA256
)

# List session types
types = client.list_session_types()

# Get session type details
session_type = client.get_session_type("support_session")

# Update session type
updated_type = client.update_session_type(
    "support_session",
    name="Updated Support Type",
    description="Updated description"
)

# Delete session type
client.delete_session_type("support_session")
```

### Monitoring & Analytics

```python
# Monitor agent activity
agent_stats = client.monitor_agent(
    "agent123",
    time_range={
        "start_time": "2024-01-01T00:00:00Z",
        "end_time": "2024-01-31T23:59:59Z"
    }
)

# Monitor session activity
session_stats = client.monitor_session("session123")
```

## Troubleshooting

### Common Issues

#### "Authentication failed" or 403 errors
- **Problem**: Your `cluster_api_key` in the SDK doesn't match the `CLUSTER_API_KEY` in your server
- **Solution**: Make sure both use the exact same value

#### "Connection refused" or "Server not reachable"
- **Problem**: Your Eion server isn't running
- **Solution**: Run `docker-compose up -d` or check your server logs

#### "Health check failed"
- **Problem**: Server is running but not responding properly
- **Solution**: Check server logs with `docker-compose logs eion-server`

### Verifying Your Setup

```bash
# 1. Check if server is running
curl http://localhost:8080/health

# 2. Test with your API key
curl -H "Authorization: Bearer my-secret-api-key-123" http://localhost:8080/health
```

## Configuration

### Environment Variables

```bash
export EION_BASE_URL="http://localhost:8080"
export EION_CLUSTER_API_KEY="my-secret-api-key-123"
```

### Configuration File

```python
client = EionClient(config_file="eion.yaml")
```

## Features

- **Cluster Management**: User, agent, and session management
- **Agent Registration**: Register and manage AI agents with permissions
- **Session Management**: Create and manage conversation sessions
- **Agent Groups**: Organize agents into teams
- **Session Types**: Define session templates with agent group assignments
- **Monitoring & Analytics**: Track agent performance and collaboration
- **Health Checks**: Monitor system health and connectivity
- **Structured Error Handling**: Comprehensive exception types
- **Authentication**: Multiple authentication methods
- **Type Hints**: Full type annotation support

## Documentation

- **Full API Documentation**: [docs/openapi.yaml](docs/openapi.yaml)
- **Agent API Guide**: [docs/agent-api-guide.json](docs/agent-api-guide.json)
- **Examples**: [example/](example/)

## Support

- **Issues**: [GitHub Issues](https://github.com/eiondb/eion-sdk-python/issues)

## License

This project is licensed under the GNU Affero General Public License v3.0 - see the [LICENSE.md](LICENSE.md) file for details.

---

Happy building with Eion! 🚀 