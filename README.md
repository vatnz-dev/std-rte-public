# Navigraph NZZC STD RTE Access Repository

This repository exists to enable Navigraph (inc. SimBrief) to routinely pull NZZC STD RTEs and import them for use within the SimBrief flight planning software.

**VATNZ Point of Contact**: Tom K, Airspace Manager.
**Navigraph/SimBrief Point of Contact**: Derek M, SimBrief Lead.

----

# Schema Outline

This JSON file contains information about New Zealand's Standard Flight Routes. This data is structured in two main sections: `metadata`, and `routes`.

Any change to this schema will be communicated to Navigraph/SimBrief with at least 30 days notice.

## File Name

The file shall follow the format: `stdRoutes.json`.

## metadata

This section contains metadata about the file itself, as well as some additional descriptive parameters for the data contained within.

There are three main subsections: `cycleInfo`, `validityNumber` and a friendly file `description`.

### cycleInfo

This section contains information about the AIRAC cycle that the data is valid for. It contains the following keys:
- `cycle` (number): The AIRAC cycle number of effectivity.
- `releaseAnrAligned` (boolean): Defines whether the release is aligned with the 56-day ANR cycle or not. If it is, it may include new ICAO 5LNC waypoints.
- `releaseType` (string): The type of release. Can be `fix`, `routine` or `schemaUpdate`.
- `fileGeneratedAt` (string): The date and time the file was generated. This should be in ISO 8601 format.

## routes

This section contains the Standard Routes Catalogue (SRC) data. The SRC is a collection of routes that are used by air traffic controllers to provide a standard route for aircraft to follow. These routes are used to reduce the workload on pilots and controllers by providing a standard route that can be used by all aircraft.

The SRC data is contained in the `routes` object. Within that object, there are multiple keys, each representing a departure aerodrome's ICAO identifier. The value of each key is an array of route objects departing from that aerodrome.

### Route Object

Within each route object, there are several keys:
- `id` (string): The route's ID (eg `WNAA1`)
- `departureAerodrome` (string): The departure aerodrome's code.
- `destinationAerodrome` (string): The destination aerodrome's code.
- `lastUpdated` (string): The AIRAC cycle that the route was last updated. Note, this is not presently used, but will be implemented in the future.
- `runwayDependant` (boolean): A boolean indicating whether the route is runway dependant or not.
- `routePoints` (array): An array of route waypoints and airways.
  - If `runwayDependant` is set to `false`, this will be an array of strings, each being a waypoint or airway.
  - if `runwayDependant` is set to `true`, this will be an array of objects, each object being a runway-dependant route.
    - In each returned object, there will be two strings: `runway` and `points`.
    - `runway` will be a string representing the aerodrome and runway that the route is dependant on.
    - `points` will be an array of strings, each being a waypoint or airway.
- `remarks` (object): An object containing additional information about the route. If there are no remarks for the route, this will be empty.


## Remark Types

The `remarks` object will contain any applicable route limitations or notes. There can be multiple properties, with each being a key-value pair. The key is the broad category and the value being the explicit limitation string.

#### `alt_`: (Altitude)
- `alt_maintain`: Indicates the altitude that the aircraft should maintain. Example: `Maintain FL240`
- `alt_mfa`: Indicates the Minimum Flight Altitude (MFA). Example: `MFA FL180`
- `alt_limit`: Indicates the altitude limit for the aircraft. Example: `9000 ft and below`
- `alt_reachBy`: Indicates the level that the aircraft should reach by a specific point. Example: `Reach FL180 by XXXXX`

#### `comm_`: (Communications)
- `comm_clrReq`: Indicates that the aircraft must accept a specific clearance. Example: `ACFT must accept a clearance and fly a published SID via XXXXX`

#### `nav_`: (Navigation)
- `nav_rnpReq`: Indicates that the aircraft requires a specific Required Navigation Performance (RNP). Example: `RNP1 required`
- `nav_otherReq`: Indicates other navigation requirements for the aircraft. Example: `RNAV1 required`

#### `env_` (Environment)
- `env_volcanicRoute`: Indicates that the route has potential hazardous flight conditions, such as volcanic activity. Example: `Hazardous flight conditions due to volcanic activity`

#### `ac_` (Aircraft)
- `ac_type`: Indicates that the route is only applicable for that type of aircraft, and can be `Prop` or `Jet`. The absence of this remark indicates that it can be used by either type.

#### `misc_` (Miscellaneous)
- `misc`: Includes any other miscellaneous remarks, and is a fallback for when a remark doesn't fit any other section. There can be multiple of these in a remarks section. Example: `For use when G351 is active`


## Examples

### Non-Runway Dependant Route

```json
{
  "id": "AACH1",
  "departureAerodrome": "NZAA",
  "destinationAerodrome": "NZCH",
  "lastUpdated": null,
  "runwayDependant": false,
  "routePoints": [
    "H384",
    "NP",
    "H252",
    "NS",
    "Y288"
  ],
  "remarks": {
    "ac_type": "Jets",
    "alt_limit": "FL240 and above"
  }
}
```

In this example, the route with ID `AACH1` departs from aerodrome `NZAA` and arrives at `NZCH`. The route is not runway dependant (`runwayDependant: false`) and follows the waypoints and airways specified in the `routePoints` array. The remarks object indicates that this route is for Jets and the altitude limit is FL240 and above.

### Runway Dependant Route

```json
{
  "id": "AAKT1",
  "departureAerodrome": "NZAA",
  "destinationAerodrome": "NZKT",
  "lastUpdated": null,
  "runwayDependant": true,
  "routePoints": [
    {
      "runway": "AA RWY 23L",
      "points": [
        "G591"
      ]
    },
    {
      "runway": "AA RWY 05R",
      "points": [
        "Q520",
        "APABO",
        "Q659",
        "PERAS",
        "G591"
      ]
    }
  ],
  "remarks": {}
}
```

In this example, the route with ID `AAKT1` departs from aerodrome `NZAA` and arrives at `NZKT`. The route is runway dependant (`runwayDependant: true`) and has different waypoints and airways depending on the runway. For runway `AA RWY 23L`, the route follows `G591`. For runway `AA RWY 05R`, the route follows the waypoints and airways specified in the corresponding `points` array. The `remarks` object is empty, indicating there are no additional remarks for this route.

