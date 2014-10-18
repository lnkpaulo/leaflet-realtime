# Leaflet Realtime

Put realtime data on a Leaflet map: live tracking GPS units, sensor data or just about anything.

## Example

Checkout the [Leaflet Realtime Demo](http://www.liedman.net/leaflet-realtime). Basic example:

```javascript
var map = L.map('map'),
    realtime = L.realtime({
        url: 'https://wanderdrone.appspot.com/',
        crossOrigin: true,
        type: 'json'
    }, {
        interval: 3 * 1000
    }).addTo(map);

realtime.on('newdata', function() {
    map.fitBounds(realtime.getBounds(), {maxZoom: 3});
});
```

## Usage

### Overview

Leaflet Realtime reads and displays GeoJSON from a provided source. A "source" is usually a URL, and data can be fetched using AJAX (XHR), JSONP. Alternatively, you can write your own source, to provide data in just about any way you want.

To be able to figure out when new features are added, when old features are removed, and which features are just updated, Leaflet Realtime needs to identify each feature uniquely. This is done using a _feature id_. Usually, this can be done using one of the feature's `properties`. By default, Leaflet Realtime will try to look for a called property `id` and use that.

Technically, `L.Realtime` extends `L.GeoJSON`, with the additional functionality that it automatically updates the data with the result from a periodic HTTP request. You can basically do anything you can do with `L.GeoJSON` with `L.Realtime` - styling, `onEachFeature`, gettings bounds, etc.

Typical usage involves instantiating `L.Realtime` with options for [`style`](http://leafletjs.com/reference.html#geojson-style) and/or [`onEachFeature`](http://leafletjs.com/reference.html#geojson-oneachfeature), to customize styling and interaction, as well as adding a listener for the [`update`](#event-update) event, to for example list the features currently visible in the map.

For ease of use and flexibility, Leaflet Realtime uses [reqwest](https://github.com/ded/reqwest) for getting data over HTTP(S), and it is bundled in the distribution.

### API

#### L.Realtime

This is a realtime updated layer that can be added to the map. It extends [L.GeoJSON](http://leafletjs.com/reference.html#geojson).

##### Creation

Factory                | Description
-----------------------|-------------------------------------------------------
`L.Realtime(<`[`Source`](#source)`> source, <`[`RealtimeOptions`](#realtimeoptions)`> options?)` | Instantiates a new realtime layer with the provided source and options

##### <a name="realtimeoptions"></a> Options

Provides these options, in addition to the options of [`L.GeoJSON`](http://leafletjs.com/reference.html#geojson).

Option                 | Type                | Default       | Description
-----------------------|---------------------|----------------------|---------------------------------------------------------
`start`                | `Boolean`           | `true`        | Should automatic updates be enabled when class is instantiated
`interval`             | `Number`            | 60000         | Automatic update interval, in milliseconds
`getFeatureId(<GeoJSON> featureData)`         | `Function`          | Returns `featureData.properties.id` | Function used to get an identifier uniquely identify a feature over time
`updateFeature(<GeoJSON> featureData, <ILayer> oldLayer, <ILayer> newLayer)`                 | `Function` | Special | Used to update an existing feature's layer; by default, points (markers) are updated, other layers are discarded and replaced with a new, updated layer. Allows to create more complex transitions, for example, when a feature is updated |

##### Events

Event         | Data           | Description
--------------|----------------|---------------------------------------------------------------
`update`      | [`UpdateEvent`](#updateevent) | Fires when the layer's data is updated

##### Methods

Method                 | Returns        | Description
-----------------------|----------------|-----------------------------------------------------------------
`start()`              | `this`         | Starts automatic updates
`stop()`               | `this`         | Stops automatic updates
`isRunning()`          | `Boolean`      | Tells if automatic updates are running
`update()`             | `this`         | Updates the layer's data (asynchronously)
`getLayer(<FeatureId> featureId)` | `ILayer` | Retrieves the layer used for a certain feature

#### <a name="source"></a> Source

The source can either be an options object that is passed to [reqwest](https://github.com/ded/reqwest) for fetching the data, or a function in case you need more freedom.

In case you use a function, the function should take two callbacks as arguments: `fn(success, error)`, with the callbacks:

* a success callback that takes GeoJSON as argument: `success(<GeoJSON> features)`
* an error callback that should take an error object and an error message (string) as argument: `error(<Object> error, <String> message)`

#### <a name="updateevent"></a> UpdateEvent

An update event is fired when the layer's data is updated from its source. The data included loosely resembles D3's [join semantics](http://bost.ocks.org/mike/join/), to make it easy to handle new features (the _enter_ set), updated features (the _update_ set) and removed features (the _exit_ set).

property      | type       | description
--------------|------------|-----------------------------------
`features`    | Object     | Complete hash of current features, with feature id as key
`enter`       | Object     | Features added by this update, with feature id as key
`update`      | Object     | Existing features updated by this update, with feature id as key
`exit`        | Object     | Existing features that were removed by this update, with feature id as key