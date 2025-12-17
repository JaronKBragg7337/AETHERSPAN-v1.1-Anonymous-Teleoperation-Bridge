# AETHERSPAN v1.1 — Anonymous Teleoperation Bridge

AETHERSPAN is a local-first, anonymous WebSocket bridge for real-time teleoperation. It maps controller input to a device adapter, returns state/receipts in real time, and logs activity into an append-only, hash-chained SQLite ledger. It also supports optional peer messaging, optional sync, and optional usage-based billing (credits).

## Key Features

- **Anonymous Sessions**: Session IDs are hashed by default; no human identity is stored unless you enable raw storage.
- **Append-Only Ledger**: Deferred batch commits to SQLite (WAL mode) with content-hash chaining for tamper-evident history.
- **Device-Agnostic**: Swap `SimulatedKinematicsAdapter` for any target (ROS, serial, HTTP, SDK, etc.).
- **Replay Mode**: On disconnect, can replay the last N teleop frames as “autonomy” mode (optional).
- **Peer Networking**: HMAC-signed peer messages between AETHERSPAN instances (optional).
- **Sync (Optional)**: Pull/push blocks between instances with an inbox world per peer.
- **Billing (Optional)**: Meter/enforce/off modes using wallet credit balances (frames, log KB, peer messages, sync blocks).
- **Runtime Config**: Toggle recording/replay/peers/billing live via `/api/config`.
- **Default Controller Mapping**: Left stick (vx/vy), right stick (wyaw/pitch), triggers (grippers). You can also send `device_cmd` directly.

## Quick Start

### Install
```bash
pip install fastapi uvicorn

Optional environment

export AETHERSPAN_SESSION_SALT="your-salt"
export AETHERSPAN_ADMIN_KEY="your-admin-key"

Run

python aetherspan.py --db aetherspan.db --host 0.0.0.0 --port 8000

Connect a client (example)

const ws = new WebSocket("ws://localhost:8000/ws/teleop/my_session");
ws.onopen = () => ws.send(JSON.stringify({
  left_stick_x: 0.5,
  left_stick_y: 0.3,
  right_stick_x: -0.2,
  triggers: { L: 0.8, R: 0.0 }
}));
ws.onmessage = (e) => console.log(JSON.parse(e.data));

HTTP Endpoints
	•	GET /api/status
	•	GET /api/config
	•	POST /api/config (patch config values)
	•	GET /api/sessions

Peers:
	•	GET /api/peers
	•	POST /api/peers (admin header required if AETHERSPAN_ADMIN_KEY is set)
	•	POST /api/peer/send
	•	POST /peer/message

Sync:
	•	GET /sync/heads
	•	GET /sync/blocks?since_ts=...
	•	POST /sync/blocks

Billing:
	•	GET /api/billing/config
	•	POST /api/billing/config (admin)
	•	GET /api/billing/wallet/{account_id}
	•	POST /api/billing/credit (admin)

Architecture
	•	DeviceAdapter: target interface (apply(cmd) -> (state, receipt)).
	•	RingBuffer: in-memory frame history (teleop/autonomy/peer/sync).
	•	DeferredCommitter: async batch writer to the append-only ledger.
	•	Persistence: SQLite tables for blocks, sessions, peers, wallets, usage.
	•	BridgeCore: WebSocket session logic, optional replay, optional peer/sync, optional billing.

Security Notes (Important)
	•	Keep shared_key values secret (peer HMAC).
	•	If you set AETHERSPAN_ADMIN_KEY, treat it like a root key.
	•	Do not expose the server publicly without TLS + firewall/VPN.
	•	The ledger is tamper-evident, not “tamper-proof” (SQLite can be modified if an attacker has disk access).

License

MIT (add a LICENSE file in the repo)
https://jaronkbragg7337.github.io/persistent-memory-substrate/
