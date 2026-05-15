# WebSocket Events

OrbitStream Backend exposes a Socket.io WebSocket server for real-time stream updates.

**URL:** `wss://api.orbitstream.xyz`

---

## Connecting

```typescript
import { io } from 'socket.io-client';

const socket = io('wss://api.orbitstream.xyz', {
  auth: { token: 'your-jwt-token' },
  transports: ['websocket'],
});

socket.on('connect', () => {
  console.log('Connected:', socket.id);
});
```

---

## Client → Server Events

### `subscribe_stream`

Subscribe to live updates for a specific stream.

```typescript
socket.emit('subscribe_stream', 'stream-uuid-here');
// Response: { event: 'subscribed', streamId: 'stream-uuid-here' }
```

### `unsubscribe_stream`

Stop receiving updates for a stream.

```typescript
socket.emit('unsubscribe_stream', 'stream-uuid-here');
// Response: { event: 'unsubscribed', streamId: 'stream-uuid-here' }
```

---

## Server → Client Events

### `stream_update`

Emitted when any field on the stream changes (claim, status update, top-up).

```typescript
socket.on('stream_update', (data) => {
  // data.streamId — the stream that changed
  // data.*        — updated fields
  console.log(data);
});
```

### `claimed`

Emitted when tokens are successfully claimed.

```typescript
socket.on('claimed', ({ streamId, amount, recipient }) => {
  console.log(`${amount} tokens claimed by ${recipient} from stream ${streamId}`);
});
```

### `status_change`

Emitted when a stream is paused, resumed, or cancelled.

```typescript
socket.on('status_change', ({ streamId, status }) => {
  // status: 'active' | 'paused' | 'cancelled' | 'completed'
  console.log(`Stream ${streamId} is now ${status}`);
});
```

---

## Full Example

```typescript
import { io } from 'socket.io-client';

const socket = io('wss://api.orbitstream.xyz');

// Subscribe on connect
socket.on('connect', () => {
  socket.emit('subscribe_stream', 'my-stream-id');
});

// Live balance update
socket.on('stream_update', (data) => {
  updateUI(data);
});

// Claim notification
socket.on('claimed', ({ amount }) => {
  showToast(`Claimed ${amount} tokens`);
});

// Handle disconnect
socket.on('disconnect', () => {
  console.log('Disconnected from OrbitStream WS');
});
```

---

## Building a live counter

```typescript
// Poll claimable every second using the formula directly
let earned = 0;
const startTime = stream.startTime;
const rate = stream.ratePerSecond;
const claimed = stream.totalClaimed;

setInterval(() => {
  const now = Math.floor(Date.now() / 1000);
  const activeSecs = now - startTime - stream.totalPausedSeconds;
  earned = activeSecs * rate;
  const claimable = Math.max(0, earned - claimed);
  setDisplayBalance(claimable);
}, 100); // update every 100ms for smooth animation
```
