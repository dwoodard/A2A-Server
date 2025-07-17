# A2A Server - Agent-to-Agent Communication Platform

## What is this?

**In simple terms:** This is a server that lets AI agents talk to each other and work together.

Imagine you have different AI assistants - one that's good at weather, another that's great at math, and one that excels at writing emails. Instead of each one working in isolation, this A2A Server lets them:

- **Find each other** - "Hey, is there an agent that can help with weather?"
- **Ask for help** - "Weather agent, what's the temperature in New York?"
- **Get real-time updates** - "Let me know when you're done processing that data"
- **Work together** - "Math agent, calculate this, then Email agent, send the results"

Think of it like a **switchboard operator for AI agents** - it handles all the communication, coordination, and collaboration between different AI services so they can work as a team instead of alone.

**Real-world example:** You ask your personal assistant to "Plan a trip to Paris." Instead of one AI trying to do everything, your assistant can:

1. Ask the Weather Agent for Paris weather
2. Request the Travel Agent to find flights
3. Have the Hotel Agent search for accommodations
4. Get the Email Agent to send you a summary

All while you get real-time updates on the progress.

## Technical Overview

The A2A Server is a Laravel-based implementation of the Agent-to-Agent (A2A) communication protocol. It provides a robust, scalable platform for building autonomous agents that can communicate, collaborate, and delegate tasks to each other in real-time.

## What is A2A?

Agent-to-Agent (A2A) is a standardized communication protocol that enables AI agents to:

- Discover each other's capabilities
- Send tasks and receive results
- Stream real-time updates during task execution
- Handle complex multi-turn conversations
- Manage asynchronous workflows with push notifications

Think of it as a "REST API for AI agents" - but with advanced features like streaming, push notifications, and standardized skill discovery.

## Core Features

### 🤖 Agent Discovery

- **Agent Cards**: Agents advertise their capabilities via standardized JSON at `/.well-known/agent.json`
- **Skill Registry**: Automatic discovery and registration of agent skills
- **Capability Negotiation**: Agents can determine what other agents can do

### 📡 Communication Protocols

- **JSON-RPC 2.0**: Standardized request/response format
- **Server-Sent Events (SSE)**: Real-time streaming of task updates
- **Push Notifications**: Webhook-based async updates
- **Multi-turn Conversations**: Stateful dialogue management

### 🛠️ Task Management

- **Lifecycle Tracking**: Tasks progress through states (submitted → working → completed)
- **Artifacts**: File and data outputs from task execution
- **History**: Complete conversation and execution history
- **Cancellation**: Ability to cancel running tasks

### 🔧 Extensible Skills System

- **Plugin Architecture**: Easy-to-implement skill interface
- **Auto-discovery**: Skills are automatically registered
- **Metadata**: Rich skill descriptions with input/output schemas
- **Closure Support**: Simple inline skill definitions

## Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Agent  │    │   A2A Server    │    │  External APIs  │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │ Agent Card  │ │    │ │ Agent Card  │ │    │ │   Weather   │ │
│ │ Discovery   │ │    │ │ Discovery   │ │    │ │     API     │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │  JSON-RPC   │◄────►│  JSON-RPC   │ │    │ │  Database   │ │
│ │   Client    │ │    │  Controller │ │    │ │   Service   │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
│                 │    │                 │    │                 │
│ ┌─────────────┐ │    │ ┌─────────────┐ │    │ ┌─────────────┐ │
│ │ SSE Stream  │◄────►│ SSE Stream  │ │    │ │   Email     │ │
│ │   Handler   │ │    │   Handler   │ │    │ │   Service   │ │
│ └─────────────┘ │    │ └─────────────┘ │    │ └─────────────┘ │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Skill System  │
                       │                 │
                       │ ┌─────────────┐ │
                       │ │    Echo     │ │
                       │ │    Skill    │ │
                       │ └─────────────┘ │
                       │                 │
                       │ ┌─────────────┐ │
                       │ │  Weather    │ │
                       │ │   Skill     │ │
                       │ └─────────────┘ │
                       │                 │
                       │ ┌─────────────┐ │
                       │ │ Custom App  │ │
                       │ │   Skills    │ │
                       │ └─────────────┘ │
                       └─────────────────┘
```

## Getting Started

### Prerequisites

- PHP 8.2+
- Laravel 12+
- Redis (for queues and caching)
- MySQL/PostgreSQL (for task storage)

### Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/dwoodard/A2A-Server.git
   cd A2A-Server
   ```

2. **Install dependencies**

   ```bash
   composer install
   npm install
   ```

3. **Publish A2A package configuration**

   ```bash
   php artisan vendor:publish --tag=a2a
   ```

4. **Configure environment**

   ```bash
   cp .env.example .env
   php artisan key:generate
   ```

5. **Setup database**

   ```bash
   php artisan migrate
   ```

6. **Configure your agent** (in `.env`)

   ```env
   A2A_AGENT_NAME="My A2A Agent"
   A2A_AGENT_DESCRIPTION="A powerful AI agent for task automation"
   A2A_AGENT_VERSION="1.0.0"
   A2A_AGENT_PROVIDER="Your Company"
   A2A_AGENT_STREAMING=true
   A2A_AGENT_PUSH=true
   ```

### Basic Usage

#### 1. Test Agent Discovery

```bash
curl http://localhost:8000/.well-known/agent.json
```

This returns your agent's capabilities:

```json
{
  "name": "My A2A Agent",
  "description": "A powerful AI agent for task automation",
  "version": "1.0.0",
  "url": "http://localhost:8000/a2a",
  "capabilities": {
    "streaming": true,
    "pushNotifications": true
  },
  "skills": [
    {
      "id": "echo",
      "name": "Echo",
      "description": "Echoes back the user message"
    }
  ]
}
```

#### 2. Send a Task

```bash
curl -X POST http://localhost:8000/a2a \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "tasks/send",
    "params": {
      "id": "task-123",
      "message": {
        "role": "user",
        "content": [{"type": "text", "text": "Hello, world!"}]
      }
    }
  }'
```

#### 3. Stream Real-time Updates

```bash
curl -X POST http://localhost:8000/a2a/subscribe \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "tasks/sendSubscribe",
    "params": {
      "id": "task-456",
      "message": {
        "role": "user",
        "content": [{"type": "text", "text": "Process this data"}]
      }
    }
  }'
```

## Creating Custom Skills

### 1. Create a Skill Class

```php
<?php

namespace App\A2A\Skills;

use Dwoodard\A2aLaravel\Skills\SkillInterface;
use Dwoodard\A2aLaravel\Models\Task;
use Dwoodard\A2aLaravel\Models\Message;

class WeatherSkill implements SkillInterface
{
    public function id(): string
    {
        return 'weather';
    }

    public function name(): string
    {
        return 'Weather Information';
    }

    public function description(): string
    {
        return 'Get current weather for a location';
    }

    public function inputSchema(): array
    {
        return [
            'type' => 'object',
            'properties' => [
                'location' => ['type' => 'string', 'description' => 'City name']
            ],
            'required' => ['location']
        ];
    }

    public function outputSchema(): array
    {
        return [
            'type' => 'object',
            'properties' => [
                'temperature' => ['type' => 'number'],
                'condition' => ['type' => 'string'],
                'location' => ['type' => 'string']
            ]
        ];
    }

    public function execute(Task $task, Message $inputMessage)
    {
        $content = $inputMessage->content;
        $location = $this->extractLocation($content);
        
        // Call weather API
        $weather = $this->getWeatherData($location);
        
        return [
            'temperature' => $weather['temp'],
            'condition' => $weather['condition'],
            'location' => $location
        ];
    }

    private function extractLocation($content): string
    {
        // Extract location from message content
        // Implementation depends on your content format
        return 'New York'; // Simplified
    }

    private function getWeatherData(string $location): array
    {
        // Call external weather API
        // Return weather data
        return [
            'temp' => 72,
            'condition' => 'Sunny'
        ];
    }
}
```

### 2. Register the Skill

In `config/a2a.php`:

```php
'skills' => [
    'echo' => Dwoodard\A2aLaravel\Skills\EchoSkill::class,
    'weather' => App\A2A\Skills\WeatherSkill::class,
    
    // Or define inline
    'calculator' => [
        'name' => 'Calculator',
        'description' => 'Performs basic math operations',
        'handler' => function(Task $task, Message $input) {
            // Simple calculation logic
            return eval($input->content);
        },
    ],
],
```

## Advanced Features

### Multi-Agent Communication

Your agent can communicate with other A2A agents:

```php
use Dwoodard\A2aLaravel\Services\AgentService;

// Discover another agent
$agentCard = app(AgentService::class)->discoverAgent('https://other-agent.com');

// Send a task to another agent
$result = app(AgentService::class)->sendTaskToAgent(
    'https://other-agent.com/a2a',
    'task-789',
    ['role' => 'user', 'content' => [['type' => 'text', 'text' => 'Process this']]],
    ['pushNotification' => ['url' => 'https://your-agent.com/a2a/push']]
);
```

### Real-time Streaming

Clients can subscribe to real-time updates:

```javascript
const eventSource = new EventSource('http://localhost:8000/a2a/subscribe', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        jsonrpc: '2.0',
        id: '1',
        method: 'tasks/sendSubscribe',
        params: {
            id: 'task-stream',
            message: {
                role: 'user',
                content: [{ type: 'text', text: 'Stream this task' }]
            }
        }
    })
});

eventSource.onmessage = function(event) {
    const update = JSON.parse(event.data);
    console.log('Task update:', update);
    
    if (update.result.final) {
        eventSource.close();
    }
};
```

### Task Lifecycle Management

```php
use Dwoodard\A2aLaravel\Services\TaskManager;

$taskManager = app(TaskManager::class);

// Create a task
$task = $taskManager->createTask([
    'id' => 'custom-task',
    'session_id' => 'session-123',
    'message' => ['role' => 'user', 'content' => 'Task content'],
    'metadata' => ['priority' => 'high']
]);

// Update task state
$taskManager->markWorking($task);
$taskManager->markCompleted($task, 'Task completed successfully');

// Cancel a task
$taskManager->cancelTask($task);
```

## Use Cases

### 1. **AI Agent Orchestration**

- Coordinate multiple specialized agents
- Build complex workflows across services
- Handle agent failover and load balancing

### 2. **Microservices Communication**

- Replace REST APIs with rich, stateful communication
- Stream real-time updates between services
- Handle long-running operations elegantly

### 3. **Automation Workflows**

- Build chains of automated tasks
- Handle approvals and human-in-the-loop processes
- Integrate with external systems and APIs

### 4. **Real-time Collaboration**

- Enable multiple agents to work on shared tasks
- Broadcast updates to interested parties
- Maintain conversation history and context

## Configuration

Key configuration options in `config/a2a.php`:

```php
return [
    'name' => env('A2A_AGENT_NAME', 'Laravel A2A Agent'),
    'description' => env('A2A_AGENT_DESCRIPTION', 'A2A-compliant agent'),
    'version' => env('A2A_AGENT_VERSION', '1.0.0'),
    'provider' => env('A2A_AGENT_PROVIDER', 'Your Company'),
    
    'capabilities' => [
        'streaming' => env('A2A_AGENT_STREAMING', true),
        'pushNotifications' => env('A2A_AGENT_PUSH', true),
    ],
    
    'authentication' => [
        'schemes' => [], // ['ApiKey', 'OAuth2']
    ],
    
    'skills' => [
        // Your skills here
    ],
];
```

## API Reference

### Core Endpoints

- `GET /.well-known/agent.json` - Agent discovery
- `POST /a2a` - JSON-RPC endpoint for all operations
- `POST /a2a/subscribe` - Server-Sent Events streaming
- `POST /a2a/push/{taskId}` - Push notification webhook

### JSON-RPC Methods

- `tasks/send` - Send a task for execution
- `tasks/sendSubscribe` - Send a task with streaming
- `tasks/get` - Get task status and results
- `tasks/cancel` - Cancel a running task
- `tasks/resubscribe` - Resume streaming for a task
- `tasks/pushNotification/set` - Configure push notifications
- `tasks/pushNotification/get` - Get push configuration

## Contributing

1. Fork the repository
2. Create a feature branch
3. Implement your changes
4. Add tests for new functionality
5. Submit a pull request

## License

This project is licensed under the MIT License. See the LICENSE file for details.

## Support

- **Documentation**: Check the `docs/` directory for detailed guides
- **Issues**: Report bugs and feature requests on GitHub
- **Community**: Join our Discord/Slack for discussions

## Roadmap

- [ ] Web dashboard for task management
- [ ] Advanced authentication schemes
- [ ] Skill marketplace integration
- [ ] Performance monitoring tools
- [ ] Multi-tenant support
- [ ] Kubernetes deployment guides

---

The A2A Server provides a powerful foundation for building sophisticated agent-based systems. Whether you're creating a simple automation tool or a complex multi-agent platform, the A2A protocol and this Laravel implementation provide the tools you need to build robust, scalable solutions.
