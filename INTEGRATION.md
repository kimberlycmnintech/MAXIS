# Connecting the MAXIS prototype later

The delivered app is frontend-only. It implements and documents a transport-independent `DataSource` contract; it does not assume HTTP, WebSocket, MQTT or direct serial access. Your hardware/backend teammate supplies the transport and sensor calibration.

## Contract

The canonical definitions are in `src/types.ts`.

```ts
interface DataSource {
  subscribe(callback: (snapshot: Snapshot) => void): () => void;
  getHistory(metric: MetricKey, range: '1h' | '6h' | '24h'): HistoryPoint[];
  sendCommand(command: Command): Promise<CommandResult>;
}
```

`subscribe` delivers a complete latest snapshot and subsequent updates. Return an unsubscribe callback. Snapshot contains installation ID, ISO timestamp, source, source availability, stop latch, connection status, equipment status, readings and recent events. Use UTC ISO timestamps; the frontend displays Malaysia time.

Each reading includes `installationId`, `equipmentId`, `metric`, `value`, `unit`, `quality` and `timestamp`. `quality` is `valid`, `stale` or `unavailable`. Zero is a valid value; missing readings use null. Stale readings retain the timestamp of the last actual measurement. Do not refresh a stale reading's timestamp just because a heartbeat arrives.

`getHistory` is synchronous in this frontend interface: an adapter should cache fetched/streamed history before returning it. History is retained by metric and filtered to the requested range. If the transport retrieves history asynchronously, implement that fetch in the adapter and notify subscribers when its cache updates. Chart aggregation is by minute, with null gaps for invalid measurements.

Commands:

```ts
type Command =
  | { type: 'source'; source: 'rainwater' | 'mains' }
  | { type: 'stop' }
  | { type: 'reset' };
```

A result includes ID, type, timestamp, status (`pending`, `confirmed`, `rejected`, `timeout`) and a human-readable message. The demo stores pending state before awaiting confirmation. The real adapter must report controller confirmation; sending a request alone must not change confirmed equipment/source state.

## Metric identifiers and units

| Group | Identifiers | Units |
| --- | --- | --- |
| Storage | tankLevel, tankVolume | %, L |
| Membrane quality | turbidityIn, turbidityOut | NTU |
| Pressure | pressureIn, pressureOut, pressureDelta | bar |
| Supply | flow | L/min |
| Source bulk meters | rainUsage, mainsUsage | m³ |
| Power | solarPower, pumpPower, uvPower, miscPower, totalPower | W |
| Cumulative energy | solarEnergy, pumpEnergy, uvEnergy, miscEnergy, totalEnergy | kWh |
| Mains measurements | mainsTurbidity, mainsPressure | NTU, bar |
| Connection | heartbeat | ms round-trip latency |

Equipment IDs: `tank`, `pump`, `membrane`, `uv`, `solar`, `mains`, `homes`. Status values: `normal`, `warning`, `critical`, `offline`, `stopped`.

## Integration boundary

`dataSource` currently exports the simulator (`MaxisStore`). `useMaxis` reads that store so local workflow actions share one state. To connect hardware, retain the inventory/order/task/incident state and add a telemetry bridge that:

1. Subscribes to the real adapter and inserts complete snapshots into the shared store, replacing simulated snapshots.
2. Delegates history requests to the adapter/cache and preserves quality gaps.
3. Delegates control commands and records requested/confirmed/rejected/timeout results and controller events.
4. Disables the simulator tick and hides scenario/reset-demo controls in real mode.
5. Uses the same equipment IDs and unit conversions across the cards, charts, scene and resident page.

This transport/bridge is future integration work; it is not a live connection in this deliverable. Authentication/authorization, durable shared persistence and hardware interlocks belong to that integration. The current role switch is a demo experience, not an access-control boundary.

## Integration verification

Verify timestamp/quality handling, source availability, measured pressure difference, monotonic bulk meter totals, meter resets, energy totals, command acknowledgment/failure and controller disconnection. Confirm mains tank configuration and quality-verification fields when the prototype supplies them. No inferred household usage or removal efficiency should replace missing measurements.
