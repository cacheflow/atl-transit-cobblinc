# CobbLinc TypeScript client

![CobbLinc logo](assets/transit-logo.png)

> **Unofficial community project.** This repository is not affiliated with,
> endorsed by, sponsored by, or maintained by CobbLinc or Cobb County Government.
> For official transit information, visit [CobbLinc](https://www.cobbcounty.gov/transportation/cobblinc).

A small Node.js client for CobbLinc bus vehicle positions and trip updates from
GTFS Realtime feeds.

Returns decoded feed snapshots with the original GTFS Realtime fields preserved.
No API key is required.

## Getting started

Use **Node.js 24+** to run the TypeScript source directly. From this directory:

```sh
npm ci
```

Create `example.ts` alongside `app.ts`:

```ts
import BusService from './app.ts';
import type { BusFeed } from './app.ts';

const buses = new BusService();

const positions: BusFeed = await buses.getPositions();
for (const entity of positions.entity) {
  if (!entity.vehicle) continue;

  console.log({
    vehicleId: entity.vehicle.vehicle?.id,
    routeId: entity.vehicle.trip?.routeId,
    position: entity.vehicle.position,
  });
}

const updates = await buses.getUpdates();
for (const entity of updates.entity) {
  if (!entity.tripUpdate) continue;

  console.log({
    tripId: entity.tripUpdate.trip.tripId,
    stopTimeUpdates: entity.tripUpdate.stopTimeUpdate,
  });
}
```

Run it with:

```sh
node example.ts
```

Importing the module and constructing a client perform no requests. Running
`node app.ts` on its own produces no output; call a client method to fetch data.

## API

`BusService` is the default export. `BusFeed` is a TypeScript type alias for the
`FeedMessage` decoded by `gtfs-realtime-bindings`.

| Method | Returns | Request timeout |
| --- | --- | --- |
| `getPositions()` | `Promise<BusFeed>` containing vehicle positions | 20 seconds |
| `getUpdates()` | `Promise<BusFeed>` containing trip updates | 20 seconds |

Each call fetches a fresh snapshot, decodes the protobuf response, and verifies
the decoded message. The complete feed is returned, including its header and
entities. The client does not filter cancellations or normalize provider data.

Fields such as vehicle descriptors, positions, and stop-time updates may be
absent. Check optional fields before using them. Protobuf 64-bit values can be
`Long` objects rather than JavaScript numbers.

## Feed endpoints

- [Vehicle positions](https://cobb.rideralerts.com/InfoPoint/gtfs-realtime.ashx?type=vehicleposition)
- [Trip updates](https://cobb.rideralerts.com/InfoPoint/gtfs-realtime.ashx?type=tripupdate)

## Errors and limitations

Requests reject on non-2xx HTTP responses, network failures, timeouts, protobuf
decoding failures, or decoded-message validation failures. There are no automatic
retries or caches.

An HTTP 200 response does not guarantee a valid feed. The provider can return a
plain-text error with a successful status; the current client attempts to decode
that body as protobuf, which can result in an `invalid wire type` error.

The current API covers vehicle positions and trip updates. It does not include
service alerts, static schedules, historical data, or normalized arrival records.

## Development status

This package currently supports direct source usage. Its `package.json` declares
the name `cobblinc`, but the `main` entry points to `index.js`, which is not
present. There is no build script or configured TypeScript declaration output;
package-root imports are not ready for use.

The `npm test` script is a placeholder that exits with an error. An automated test
suite has not been configured.

## License

ISC, as declared in `package.json`.

The CobbLinc name and logo belong to their respective owners and are used only
to identify the transit service. Their inclusion does not imply endorsement.
The logo is sourced from the [CobbLinc bus tracker](https://cobb.rideralerts.com/InfoPoint/Content/images/logo_web.png)
and is not covered by this project's software license.
