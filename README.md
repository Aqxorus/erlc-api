# ERLC API Wrapper

A comprehensive Node.js wrapper for the Emergency Response: Liberty County (ERLC) API with advanced features like automatic rate limiting, request queuing, intelligent caching, and real-time event monitoring.

## Features

- **Type Safety**: Full TypeScript-style JSDoc types for better development experience
- **Rate Limiting**: Automatic rate limit handling with exponential backoff
- **Request Queuing**: Prevents API rate limits by queueing requests
- **Caching**: Reduces API calls with intelligent caching
- **Error Handling**: Friendly error messages with detailed error codes
- **Real-time Events**: Event subscription system for monitoring changes

## Installation

```bash
npm install erlc-api-wrapper
```

## Project Structure

```
api/
├── index.js         # Main exports and client factory functions
├── client.js        # Core ERLC API client
├── types.js         # Type definitions and error handling
├── rateLimiter.js   # Rate limiting implementation
├── queue.js         # Request queue management
├── cache.js         # Memory cache implementation
└── subscription.js  # Real-time event subscription system
```

## Quick Start

```javascript
const { newClientWithQueueAndCache } = require('erlc-api-wrapper');

// Initialize the client
const client = newClientWithQueueAndCache(
  'your-api-key-here',
  {
    workers: 2,
    interval: 200, // 200ms between requests
    ttl: 30000, // 30 second cache
  },
  {
    baseURL: 'https://api.policeroleplay.community/v1',
    timeout: 15000, // 15 second timeout
  }
);

// Use the API
const serverData = await client.getServer();
console.log('Server Info:', serverData);
```

## API Methods

The wrapper provides access to all ERLC API endpoints:

### Server Information

```javascript
// Get server status and information
const serverData = await client.getServer();

// Get server queue information
const queueData = await client.getQueue();

// Get server ban list
const bansData = await client.getBans();
```

### Player Data

```javascript
// Get all players currently on the server
const players = await client.getPlayers();
```

### Logs and History

```javascript
// Get command execution logs
const commandLogs = await client.getCommandLogs();

// Get moderator call logs
const modCalls = await client.getModCalls();

// Get kill/death logs
const killLogs = await client.getKillLogs();

// Get player join/leave logs
const joinLogs = await client.getJoinLogs();
```

### Vehicles

```javascript
// Get all vehicles on the server
const vehicles = await client.getVehicles();
```

### Command Execution

```javascript
// Execute server commands
await client.executeCommand(':announce Hello World!');
await client.executeCommand(':kick PlayerName Reason');
```

## Error Handling

The wrapper provides enhanced error handling with user-friendly messages:

```javascript
const { getFriendlyErrorMessage } = require('erlc-api-wrapper');

try {
  await client.executeCommand(':announce Test');
} catch (error) {
  const friendlyMessage = getFriendlyErrorMessage(error);
  console.error('Command failed:', friendlyMessage);
}
```

Common error scenarios handled:

- Invalid API keys
- Rate limit exceeded
- Server timeouts
- Network connectivity issues
- Invalid command syntax

## Real-time Event Monitoring

Monitor server events in real-time with the subscription system:

```javascript
const { EventType } = require('erlc-api-wrapper');

// Create a subscription for multiple event types
const subscription = client.subscribe([
  EventType.PLAYERS,
  EventType.COMMANDS,
  EventType.KILLS,
]);

// Set up event handlers
subscription.handle({
  playerHandler: (changes) => {
    console.log('Player changes detected:', changes);
  },
  commandHandler: (logs) => {
    console.log('New commands executed:', logs);
  },
  killHandler: (kills) => {
    console.log('New kills recorded:', kills);
  },
});

// Start monitoring
await subscription.start();

// Stop monitoring when done
subscription.stop();
```

### Available Event Types

- `EventType.PLAYERS` - Player join/leave events
- `EventType.COMMANDS` - Command execution events
- `EventType.KILLS` - Kill/death events
- `EventType.MODCALLS` - Moderator call events
- `EventType.JOINS` - Player join events
- `EventType.BANS` - Ban/unban events

## Configuration Options

The wrapper supports various client configurations to suit your needs:

### Basic Client

```javascript
const { newClient } = require('erlc-api-wrapper');
const client = newClient('your-api-key');
```

### Client with Queue Management

```javascript
const { newClientWithQueue } = require('erlc-api-wrapper');
const client = newClientWithQueue(
  'your-api-key',
  3, // 3 worker threads
  500 // 500ms interval between requests
);
```

### Client with Caching

```javascript
const { newClientWithCache } = require('erlc-api-wrapper');
const client = newClientWithCache(
  'your-api-key',
  60000 // 60 second cache TTL
);
```

### Full Configuration

```javascript
const { newClientWithQueueAndCache } = require('erlc-api-wrapper');
const client = newClientWithQueueAndCache(
  'your-api-key',
  {
    workers: 2, // Number of worker threads
    interval: 200, // Milliseconds between requests
    ttl: 30000, // Cache time-to-live in milliseconds
  },
  {
    baseURL: 'https://api.policeroleplay.community/v1',
    timeout: 15000, // Request timeout in milliseconds
  }
);
```

### Configuration Parameters

| Parameter  | Description                          | Default          |
| ---------- | ------------------------------------ | ---------------- |
| `workers`  | Number of concurrent request workers | 1                |
| `interval` | Milliseconds between requests        | 1000             |
| `ttl`      | Cache time-to-live in milliseconds   | 60000            |
| `baseURL`  | ERLC API base URL                    | Official API URL |
| `timeout`  | Request timeout in milliseconds      | 10000            |

## Benefits

1. **Automatic Rate Limiting**: Built-in rate limit handling with exponential backoff
2. **Request Queuing**: Prevents API overwhelm with intelligent request queuing
3. **Intelligent Caching**: Reduces redundant API calls and improves performance
4. **Enhanced Error Messages**: User-friendly error descriptions with detailed context
5. **Type Safety**: Full JSDoc type definitions for better IDE support
6. **Consistent API**: All methods follow the same pattern for predictability
7. **Real-time Monitoring**: Event subscription system for live server monitoring
8. **Future-Proof**: Designed for easy extension with new features

## Examples

### Basic Server Monitoring

```javascript
const { newClientWithQueueAndCache } = require('erlc-api-wrapper');

const client = newClientWithQueueAndCache('your-api-key');

async function monitorServer() {
  try {
    const server = await client.getServer();
    console.log(`Server: ${server.name}`);
    console.log(`Players: ${server.currentPlayers}/${server.maxPlayers}`);
    console.log(`Queue: ${server.queueCount} waiting`);
  } catch (error) {
    console.error('Failed to get server info:', error.message);
  }
}

// Monitor every 30 seconds
setInterval(monitorServer, 30000);
```

### Command Execution with Error Handling

```javascript
async function executeServerCommand(command) {
  try {
    await client.executeCommand(command);
    console.log(`Command executed successfully: ${command}`);
  } catch (error) {
    const friendlyMessage = getFriendlyErrorMessage(error);
    console.error(`Command failed: ${friendlyMessage}`);
  }
}

// Examples
await executeServerCommand(':announce Server restart in 5 minutes');
await executeServerCommand(':kick PlayerName Reason for kick');
```

### Real-time Player Monitoring

```javascript
const { EventType } = require('erlc-api-wrapper');

const subscription = client.subscribe([EventType.PLAYERS]);

subscription.handle({
  playerHandler: (changes) => {
    changes.joined.forEach((player) => {
      console.log(`${player.username} joined the server`);
    });

    changes.left.forEach((player) => {
      console.log(`${player.username} left the server`);
    });
  },
});

await subscription.start();
```

## Troubleshooting

### Common Issues

#### Authentication Errors

- Verify your API key is correct and active
- Check that the API key has the necessary permissions
- Ensure you're using the correct base URL

#### Rate Limiting

- The wrapper automatically handles rate limits, but excessive requests may still cause delays
- Consider increasing the `interval` parameter in queue configuration
- Monitor your API usage to stay within limits

#### Network Issues

- Check your internet connectivity
- Verify the ERLC API service is online
- Increase the `timeout` parameter if requests are timing out

#### Caching Issues

- Clear cache by creating a new client instance
- Adjust the `ttl` parameter for different cache durations
- Use `client.clearCache()` if available

### Getting Help

1. Check the console for detailed error messages
2. Use `getFriendlyErrorMessage()` for user-friendly error descriptions
3. Review the error codes in the API response
4. Check the ERLC API documentation for endpoint-specific issues

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Changelog

### v1.0.0

- Initial release with full ERLC API coverage
- Rate limiting and request queuing
- Intelligent caching system
- Real-time event monitoring
- Enhanced error handling
