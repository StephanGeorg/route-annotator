[![Build Status](https://travis-ci.org/mapbox/route-annotator.svg?branch=master)](https://travis-ci.org/mapbox/route-annotator) [![CodeCov](https://codecov.io/gh/mapbox/route-annotator/branch/master/graph/badge.svg)](https://codecov.io/gh/mapbox/route-annotator/branch/master)

# Route Annotator

This is a NodeJS module that indexes connected node pairs in OSM data, and allows you to query for
meta-data.  It's useful for retrieving tag information when you have geometry from your basemap.

## Requires

- Node >= 4

## Building

To install via binaries:

```
npm install
```

To install from source, first install a C++14 capable compiler then run:


```
make
```

## Testing

To run the tests (which run both the JS tests and the C++ tests):

```
make test
```

To run just the JS tests:

```
npm test
```

To run the C++ tests:

```
./build/Release/basic-tests
```

## Usage

This library contains three main modules: `Annotator`, `SegmentSpeedLookup`, and `WaySpeedLookup`

### Annotator

The `Annotator()` object is for looking up OSM tag data from OSM node IDs or coordinates.

**Example:**
```
var Annotator = require('@mapbox/route-annotator');
var path = require('path');

// Lookup some nodes and find out which ways they were on,
// and what tags they had
var taglookup = new Annotator();
taglookup.loadOSMExtract(path.join(__dirname,'data/winthrop.osm'), (err) => {
  if (err) throw err;
  var nodes = [50253600,50253602,50137292];
  taglookup.annotateRouteFromNodeIds(nodes, (err, wayIds) => {
    if (err) throw err;
    taglookup.getAllTagsForWayId(wayIds[0], (err, tags) => {
      if (err) throw err;
      console.log(tags);
    });
  });
});

var annotator = new Annotator({ coordinates: true });
// Do the same thing, but this time use coordinates instead
// of node ids.  Internally, a radius search finds the closest
// node within 5m
annotator.loadOSMExtract(path.join(__dirname,'data/winthrop.osm'), (err) => {
  if (err) throw err;
  var coords = [[-120.1872774,48.4715898],[-120.1882910,48.4725110]];
  annotator.annotateRouteFromLonLats(coords, (err, wayIds) => {
    if (err) throw err;
    annotator.getAllTagsForWayId(wayIds[0], (err, tags) => {
      if (err) throw err;
      console.log(tags);
    });
  });
});

```

### SegmentSpeedLookup

The `SegmentSpeedLookup()` object is for loading segment speed information from CSV files, then looking it up quickly from an in-memory hashtable.

**Example:**
```
var segmentspeedlookup = new (require('route_annotator')).SegmentSpeedLookup();

// Loads example.csv, then looks up the pairs 123-124, 124-125, 125-126
// and prints the speeds for those segments (3 values) as comma-separated
// data
segmentspeedlookup.loadCSV("example.csv", (err) => {
  if (err) throw err;
  segmentspeedlookup.getRouteSpeeds([123,124,125,126],(err,results) => {
    if (err) throw err;
    console.log(results.join(","));
  });
});
```

The `loadCSV` method can also be passed an array of filenames.

### WaySpeedLookup

The `WaySpeedLookup()` object is for loading way speed information from CSV files, then looking it up quickly from an in-memory hashtable.

**Example:**
```
var wayspeedlookup = new (require('route_annotator')).WaySpeedLookup();

// Loads example.csv, then looks up the ways 1111,2222,3333,4444
// and prints the speeds for those ways as comma-separated
// data
wayspeedlookup.loadCSV("example.csv", (err) => {
  if (err) throw err;
  wayspeedlookup.getRouteSpeeds([1111,2222,3333,4444],(err,results) => {
    if (err) throw err;
    console.log(results.join(","));
  });
});
```

The `loadCSV` method can also be passed an array of filenames.

---

### Example HTTP server
This will not use a tag file.

```
npm install
curl --remote-name http://download.geofabrik.de/europe/monaco-latest.osm.pbf
node example-server.js monaco-latest.osm.pbf
```

Then, in a new terminal, you should be able to do:

```
curl "http://localhost:5000/coordlist/7.422155,43.7368838;7.4230139,43.7369751"
```
---

### Release

_You must be a member of the Mapbox npm organization to do this!_

- `git checkout master`
- Update CHANGELOG.md
- Bump version in package.json, package-lock.json
- `git commit -am "vx.y.z [publish binary] | [republish binary]"` with Changelog list in commit message
- Wait for Travis to finish publishing binaries so that all travis tests pass for the publish binary commit.
- `git tag vx.y.z -a` with Changelog list in tag message
- `git push origin master; git push origin --tags`
- `npm publish --access=public`
