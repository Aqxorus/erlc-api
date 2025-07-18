# ERLC API Wrapper

## Overview

- **Type Safety**: Full TypeScript-style JSDoc types for better development experience
- **Rate Limiting**: Automatic rate limit handling with exponential backoff
- **Request Queuing**: Prevents API rate limits by queueing requests
- **Caching**: Reduces API calls with intelligent caching
- **Error Handling**: Friendly error messages with detailed error codes
- **Real-time Events**: Event subscription system for monitoring changes

## New API Structure

The new API is located in `/src/api/` with the following files:

- `index.js` - Main exports and client factory functions
- `client.js` - Core ERLC API client
- `types.js` - Type definitions and error handling
- `rateLimiter.js` - Rate limiting implementation
- `queue.js` - Request queue management
- `cache.js` - Memory cache implementation
- `subscription.js` - Real-time event subscription system

## Client Initialization

In your main `index.js`, the ERLC client is now initialized with:

```javascript
const { newClientWithQueueAndCache } = require('./api');

// Initialize ERLC API client with rate limiting and caching
const apiKey =
  client.config.debugMode === 1 ? client.config.apiKey1 : client.config.apiKey;
client.erlc = newClientWithQueueAndCache(
  apiKey,
  {
    workers: 2,
    interval: 200, // 200ms between requests
    ttl: 30000, // 30 second cache
  },
  {
    baseURL: client.config.baseURL,
    timeout: 15000, // 15 second timeout
  }
);
```

## API Methods

All ERLC API methods are now available through `client.erlc`:

### Server Information

```javascript
const serverData = await client.erlc.getServer();
const queueData = await client.erlc.getQueue();
const bansData = await client.erlc.getBans();
```

### Player Data

```javascript
const players = await client.erlc.getPlayers();
```

### Logs

```javascript
const commandLogs = await client.erlc.getCommandLogs();
const modCalls = await client.erlc.getModCalls();
const killLogs = await client.erlc.getKillLogs();
const joinLogs = await client.erlc.getJoinLogs();
```

### Vehicles

```javascript
const vehicles = await client.erlc.getVehicles();
```

### Commands

```javascript
await client.erlc.executeCommand(':announce Hello World!');
```

## Error Handling

The new wrapper provides enhanced error handling:

```javascript
const { getFriendlyErrorMessage } = require('./api');

try {
  await client.erlc.executeCommand(':announce Test');
} catch (error) {
  const friendlyMessage = getFriendlyErrorMessage(error);
  console.error('Command failed:', friendlyMessage);
}
```

## Real-time Events (Advanced)

The wrapper supports real-time event monitoring:

```javascript
const { EventType } = require('./api');

const subscription = client.erlc.subscribe([
  EventType.PLAYERS,
  EventType.COMMANDS,
  EventType.KILLS,
]);

subscription.handle({
  playerHandler: (changes) => {
    console.log('Player changes:', changes);
  },
  commandHandler: (logs) => {
    console.log('New commands:', logs);
  },
  killHandler: (kills) => {
    console.log('New kills:', kills);
  },
});

await subscription.start();
```

## Migrated Files

The following files have been updated to use the new API wrapper:

### Commands

- `/src/Commands/ERLC/bans.js` - Now uses `client.erlc.getBans()`
- `/src/Commands/ERLC/game.js` - Now uses `client.erlc.getServer()` and `client.erlc.getQueue()`
- `/src/Commands/ERLC/x.js` - Now uses `client.erlc.executeCommand()`

### Tasks

- `/src/Tasks/joinLogs.js` - Now uses `client.erlc.getJoinLogs()`
- `/src/Tasks/killLogs.js` - Now uses `client.erlc.getKillLogs()`
- `/src/Tasks/modcallLogs.js` - Now uses `client.erlc.getModCalls()`
- `/src/Tasks/reminders.js` - Now uses `client.erlc.executeCommand()`
- `/src/Tasks/statistics.js` - Now uses `client.erlc.getServer()`

### Events

- `/src/Events/Vote/sessionVoteSystem.js` - Now uses `client.erlc.getServer()`

## Benefits of the New Wrapper

1. **Automatic Rate Limiting**: No more manual rate limit handling
2. **Request Queuing**: Prevents overwhelming the API
3. **Intelligent Caching**: Reduces redundant API calls
4. **Better Error Messages**: User-friendly error descriptions
5. **Type Safety**: Better IDE support and fewer runtime errors
6. **Consistent API**: All methods follow the same pattern
7. **Future-Proof**: Easy to extend with new features

## Configuration Options

You can customize the client behavior:

```javascript
// Basic client
const client = newClient(apiKey);

// With custom queue settings
const client = newClientWithQueue(apiKey, 3, 500); // 3 workers, 500ms interval

// With custom cache settings
const client = newClientWithCache(apiKey, 60000); // 60 second cache

// Full customization
const client = newClientWithQueueAndCache(
  apiKey,
  {
    workers: 2,
    interval: 200,
    ttl: 30000,
  },
  {
    baseURL: 'https://api.policeroleplay.community/v1',
    timeout: 15000,
  }
);
```

## Old Functions Removal

The old ERLC functions in `/src/Functions/ERLC/` are no longer needed and can be safely removed:

- `getBans.js`
- `getJoinLogs.js`
- `getKillLogs.js`
- `getModcallLogs.js`
- `getQueue.js`
- `getServer.js`
- `rateLimiter.js`
- `runCommand.js`

## Testing

After migration, test the following to ensure everything works:

1. Execute a command using `/x` slash command
2. Check that game info displays correctly with `/game`
3. Verify that logs are being processed (join/leave, kills, mod calls)
4. Confirm that reminders are being sent
5. Check that the statistics channel updates correctly

## Troubleshooting

If you encounter issues:

1. Check the console for error messages
2. Verify your API key is correct in config.json
3. Ensure the base URL is correct
4. Check that your server has internet connectivity
5. Review the error details provided by `getFriendlyErrorMessage()`

The new wrapper provides much more robust error handling and should give you clear information about any issues.
