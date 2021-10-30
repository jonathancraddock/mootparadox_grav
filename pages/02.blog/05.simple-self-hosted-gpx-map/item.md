---
title: 'Simple Self-Hosted GPX Map'
visible: false
taxonomy:
    category:
        - coding
    tag:
        - code
        - javascript
        - map
        - self-hosted
routes:
    aliases:
        - /coding/test
date: '27-10-2021 12:30'
---

## Simple Self-Hosted GPX Map

My aim was simple. And perhaps it was *too* simple.

I wanted an open-source mapping solution that would allow me 
to embed a map/GPX route into a blog post in plain and simple fashion. I wanted the 
option to go full screen, and by "full screen" I meant actual full screen in a literal
sense, not a postage stamp sized map in an ugly looking box, surrounded by useless clutter.

===

### Pennine Way, Bowlees to Dufton

The map below shows a short (but very worthy) section of the Pennine Way from Bowlees 
to Dufton via Cow Green and High Cup Nick.

<p>

[View Full Screen](https://map.mootparadox.com/full/bowleesdufton) | [GPX](https://map.mootparadox.com/gpx/bowleesdufton)  
<iframe src="https://map.mootparadox.com/embed/bowleesdufton"
        height="500" width="100%" style="border:none;">
</iframe></p>

On a mobile you have to scroll using the edge of the page, and maybe that's not perfect, 
but I hate being told to
drag a map using two fingers. When I drag a map, I drag it with one finger! OK?! ;-)

### Framework

I settled on [LeafletJS](https://leafletjs.com/) with OpenTopoMap tiles, and 
[Leaflet-GPX](https://github.com/mpetazzoni/leaflet-gpx) to display the route. Ironically,
this open-source approach is so powerful that many of the best online tutorials 
are decribing quite complex
scenarios. If all you want to do is put a GPX route onto a map, you only need a few lines
of Javascript.

>NOTE: The default name of the start and end pins are `pin-icon-start.png` and `pin-icon-end.png`.
>The substring 'icon' is often blocked by products such as AdblockPlus and uBlock Origin, so it's
>advisable to rename them.

Keeping it simple, I envisaged three scenarios:

* View an embedded map
* View a full screen map
* Download the GPX file

### So How Does It Work?

You can keep the URLs tidy with a simple `.htaccess` rewrite rule.

```bash
RewriteEngine On
RewriteBase /
RewriteRule ^embed/([A-Za-z0-9-]+)/?$ index.php?map=$1&display=embed [NC,L]
RewriteRule ^full/([A-Za-z0-9-]+)/?$ index.php?map=$1&display=full [NC,L]
RewriteRule ^gpx/([A-Za-z0-9-]+)/?$ download.php?map=$1 [NC,L]
```

A GPX file named "bowleesdufton.gpx" simply ends up with the route `./gpx/bowleesdufton`, and 
similarly `./full/bowleesdufton` or `./embed/bowleesdufton` for the corresponding map.

> Remember some kind of 404 behaviour. In my implementation I cuurently have
> unroutable requests dropping the user into a default blank map. (Is that the best
> outcome?)

It seemed pointless displaying a GPX file as text in a browser. If anyone actually wants
the GPX file you would have to think their preference would be to download the file, or open it
in another app. Enter `download.php` to take care of that.

```php
<?php

$mapin = ( isset( $_GET['map'] ) ) ? $_GET["map"] . '.gpx' : 'default.gpx';
$_map = filter_var($mapin, FILTER_SANITIZE_STRING, FILTER_FLAG_STRIP_HIGH);

$file = "/PATH_HERE/gpx/".$_map;

// Warn if file does not exist
if( !file_exists($file) ) die("--Requested GPX file is not available--");

// Download
header("Content-Disposition: attachment; filename=\"" . basename($file) . "\"");
header("Content-Length: " . filesize($file));
header("Content-Type: application/octet-stream;");
readfile($file);

?>
```

This also has the advantage that the file is downloaded and you remain on the same page.

> Keep in mind you're dealing here with a *Linux* path, not the path as seen by a browser.

In `index.php` the values should be sanitised and the name of the map/GPX filename needs to 
be written into a Javascript variable. In this example `$_map` is the sanitised GPX filename
and `$_dis` is the display type. Currently only 'full' or 'embed', but 'thumbnail', 'social',
and others could perhaps be useful in the future.

```php
$mapin = ( isset( $_GET['map'] ) ) ? $_GET["map"] . '.gpx' : 'default.gpx';
$_map = filter_var($mapin, FILTER_SANITIZE_STRING, FILTER_FLAG_STRIP_HIGH);
$disin = ( isset( $_GET['display'] ) ) ? $_GET["display"] : 'embed';
$_dis = filter_var($disin, FILTER_SANITIZE_STRING, FILTER_FLAG_STRIP_HIGH);
```

The Javascript for the map is surprisingly simple.

If a GPX file is supplied then the map
sizes and centres on the route. If there's no GPX I default the centre of the map to the summit 
of Cross Fell; it's just an arbitray central location.

```javascript
// Define map and set default coordinates to Cross Fell trig point.
var mymap = L.map('mapid').setView([54.7030399,-2.4955354], 14);

// Add the specified GPX file.
new L.GPX(gpx, {
  async: true,
  marker_options: {
    startIconUrl: '/embed/pin-start.png',
    endIconUrl: '/embed/pin-end.png',
    shadowUrl: '/embed/pin-shadow.png'
  },
  polyline_options: {color: 'red', opacity: 0.7, weight: 4}
}).on('loaded', function(e) {
  mymap.fitBounds(e.target.getBounds());
}).addTo(mymap);

// Load the map tiles.
L.tileLayer('https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', {
        maxZoom: 17,
        attribution: 'Map data: &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a> (<a href="https://creativecommons.org/licenses/by-sa/3.0/">CC-BY-SA</a>)'
}).addTo(mymap);
```

My preference for OpenTopoMap meant it was better to use a red polyline for the GPX route
and I slightly increased its weight. Apart from that, it's the default code from the 
LeafletJS documentation.

### Summary

I was pleasantly surprised by how easy this is. Not only is it simple, but if
you use a CDN for the scripts (and obviously the map tiles are not stored locally) then
it's very efficient on your own bandwidth.

```markdown
[View Full Screen](https://map.mootparadox.com/full/bowleesdufton) | [GPX](https://map.mootparadox.com/gpx/bowleesdufton)  
<iframe src="https://map.mootparadox.com/embed/bowleesdufton"
    height="500" width="100%" style="border:none;">
</iframe><br />
```

To insert into a markdown document does require a line of HTML for the iframe, but
arguably I think this is a good use-case for an iframe and it reads very easily. The example
above highlights all three URL patterns.

* [./full/bowleesdufton](https://map.mootparadox.com/full/bowleesdufton)
* [./gpx/bowleesdufton](https://map.mootparadox.com/gpx/bowleesdufton)
* [./embed/bowleesdufton](https://map.mootparadox.com/embed/bowleesdufton)

### Links

Resources used in this solution:

* [LeafletJS](https://leafletjs.com/)
* [Leaflet-GPX](https://github.com/mpetazzoni/leaflet-gpx)
* [OpenTopoMap](https://opentopomap.org/#map=14/55.02982/-2.12345)