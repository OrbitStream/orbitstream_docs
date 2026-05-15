# Integration Guide

This guide shows how to integrate OrbitStream into your application using the Stellar SDK.

---

## Prerequisites

```bash
npm install @stellar/stellar-sdk socket.io-client
```

---

## 1. Create a stream (frontend)

```typescript
import {
  Contract,
  xdr,
  TransactionBuilder,
  Networks,
  SorobanRpc,
  Keypair,
} from '@stellar/stellar-sdk';

const RPC_URL    = 'https://soroban-testnet.stellar.org';
const CONTRACT   = 'YOUR_CONTRACT_ID';
const NETWORK    = Networks.TESTNET;

async function createStream(
  senderKeypair:    Keypair,
  recipientAddress: string,
  tokenContract:    string,
  ratePerSecond:    bigint,  // stroops per second
  durationSeconds:  bigint,  // 0 = open-ended
  depositAmount:    bigint,  // total stroops to lock
) {
  const server  = new SorobanRpc.Server(RPC_URL);
  const account = await server.getAccount(senderKeypair.publicKey());
  const contract = new Contract(CONTRACT);

  const tx = new TransactionBuilder(account, {
    fee: '100000',
    networkPassphrase: NETWORK,
  })
    .addOperation(contract.call(
      'create_stream',
      xdr.ScVal.scvAddress(xdr.ScAddress.scAddressTypeAccount(
        xdr.PublicKey.publicKeyTypeEd25519(
          Buffer.from(senderKeypair.rawPublicKey())
        )
      )),
      // ... recipient, token, rate, duration, deposit
    ))
    .setTimeout(30)
    .build();

  const prepared = await server.prepareTransaction(tx);
  prepared.sign(senderKeypair);

  const result = await server.sendTransaction(prepared);
  return result;
}
```

---

## 2. Claim tokens (frontend)

```typescript
async function claimStream(recipientKeypair: Keypair, streamId: number) {
  const server   = new SorobanRpc.Server(RPC_URL);
  const account  = await server.getAccount(recipientKeypair.publicKey());
  const contract = new Contract(CONTRACT);

  const tx = new TransactionBuilder(account, { fee: '100000', networkPassphrase: NETWORK })
    .addOperation(contract.call(
      'claim',
      xdr.ScVal.scvAddress(/* recipient address */),
      xdr.ScVal.scvU64(xdr.Uint64.fromString(streamId.toString())),
    ))
    .setTimeout(30)
    .build();

  const prepared = await server.prepareTransaction(tx);
  prepared.sign(recipientKeypair);
  return server.sendTransaction(prepared);
}
```

---

## 3. Read claimable amount (read-only)

```typescript
async function getClaimable(streamId: number): Promise<bigint> {
  const server   = new SorobanRpc.Server(RPC_URL);
  const contract = new Contract(CONTRACT);

  const result = await server.simulateTransaction(
    new TransactionBuilder(/* dummy account */)
      .addOperation(contract.call(
        'claimable_amount',
        xdr.ScVal.scvU64(xdr.Uint64.fromString(streamId.toString())),
      ))
      .build()
  );

  return xdr.ScVal.fromXDR(result.result!.retval, 'base64').i128().lo().toBigInt();
}
```

---

## 4. Via the REST API

If you prefer not to interact with Soroban directly, use the OrbitStream Backend:

```typescript
const API = 'https://api.orbitstream.xyz/api/v1';

// Login
const { access_token } = await fetch(`${API}/auth/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ walletAddress: 'GAXYZ...' }),
}).then(r => r.json());

const headers = {
  'Content-Type': 'application/json',
  'Authorization': `Bearer ${access_token}`,
};

// Create stream
const stream = await fetch(`${API}/streams`, {
  method: 'POST',
  headers,
  body: JSON.stringify({
    recipient:      'GBCD...7E2F',
    tokenAddress:   'CBIELTK6...',
    ratePerSecond:  347222,
    totalDeposited: 1_000_000_000,
    durationSeconds: 2_592_000,
  }),
}).then(r => r.json());

// Check claimable
const { claimable } = await fetch(`${API}/streams/${stream.id}/claimable`, { headers })
  .then(r => r.json());

console.log(`Claimable: ${claimable} stroops`);
```

---

## 5. Live balance counter (React example)

```tsx
import { useEffect, useState } from 'react';
import { io } from 'socket.io-client';

function StreamBalance({ stream }: { stream: Stream }) {
  const [claimable, setClaimable] = useState(0);

  useEffect(() => {
    // Local calculation — no server round-trip needed
    const interval = setInterval(() => {
      const now        = Math.floor(Date.now() / 1000);
      const activeSecs = now - stream.startTime - stream.totalPausedSeconds;
      const earned     = activeSecs * stream.ratePerSecond;
      setClaimable(Math.max(0, Math.min(earned - stream.totalClaimed, stream.totalDeposited - stream.totalClaimed)));
    }, 100);

    return () => clearInterval(interval);
  }, [stream]);

  // WebSocket for server-confirmed updates
  useEffect(() => {
    const socket = io('wss://api.orbitstream.xyz');
    socket.emit('subscribe_stream', stream.id);
    socket.on('claimed', () => setClaimable(0));
    return () => { socket.disconnect(); };
  }, [stream.id]);

  return (
    <div>
      <span className="font-mono text-4xl">{(claimable / 1e7).toFixed(4)}</span>
      <span className="text-gray-500 ml-2">XLM</span>
    </div>
  );
}
```

---

## Rate conversion reference

| Rate | Per second | Per minute | Per hour | Per day |
|------|-----------|-----------|---------|---------|
| 1 XLM/day | ~115,740 stroops | ~6.9M stroops | ~416.7M | 10B |
| 1 XLM/hour | ~2,778 stroops | ~166.7k | 10M | 240M |
| 1 USDC/day | ~115,740 (7 decimals) | — | — | — |

> Stellar assets use 7 decimal places. 1 XLM = 10,000,000 stroops.
