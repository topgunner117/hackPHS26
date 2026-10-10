# Address and radius → Google Solar API

Requires Python 3.9+. For reliable HTTPS certificates, install the certificate package in a local environment:

```sh
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

The script finds this environment's certificate bundle even when run with `python3 solar_address.py`.

Run in this folder:

```sh
python3 solar_address.py
```

Enter a complete street address, radius, and `feet` or `meters`. The script currently includes your local testing key as `DEFAULT_API_KEY`, so no key prompt is needed. Set `GOOGLE_MAPS_API_KEY` in your environment to override it. Remove the embedded key before sharing or committing the script. Enable the **Geocoding API** in your Google Cloud project, and enable **Solar API** if you want to fetch solar data. Google Maps Platform requires billing; API calls may incur charges.

You can also supply the inputs as arguments:

```sh
python3 solar_address.py --address "1600 Amphitheatre Parkway, Mountain View, CA 94043" --radius 300 --unit feet
```

Output is JSON with the resolved address, latitude, longitude, radius and diameter in both units, API parameters, and a copyable Solar request URL. Replace `YOUR_API_KEY` in that URL with your key. Add `--fetch-solar` to request Solar data directly and include the response under `solar_data`:

```sh
python3 solar_address.py --address "1600 Amphitheatre Parkway, Mountain View, CA 94043" --radius 100 --unit meters --fetch-solar
```

The address must resolve to the lower 48 states or Washington, DC. Alaska, Hawaii, US territories, and foreign addresses return **Not allowed**. Partial or ambiguous matches are rejected; inspect the resolved address in the output. Validation uses Google's country and state address components, rather than a rectangular US bounding box. The restriction applies to the center address; a radius near a border can extend outside the US.

A radius measures distance **from the center**, so a 300-foot radius spans a 600-foot diameter. One foot is exactly 0.3048 meters. Pass `radiusMeters` to Google's `dataLayers:get` endpoint to request the region around the coordinates; `buildingInsights:findClosest` only returns a nearby building.

Google's single-request limits:

| Radius in meters | Pixel size chosen | Layers requested |
| --- | --- | --- |
| 0 < radius ≤ 100 | 0.1 m | All layers |
| 100 < radius ≤ 175 | 0.25 m | All layers |
| 175 < radius ≤ 250 | 0.25 m | Imagery and annual flux |
| 250 < radius ≤ 500 | 0.5 m | Imagery and annual flux |
| 500 < radius ≤ 1,000 | 1 m | Imagery and annual flux |

Larger radii are rejected, not reduced. Covering them needs multiple requests and combining their raster data. Monthly flux and hourly shade are unavailable above 175 meters in one request. Solar coverage is not guaranteed for every valid US address; Google can return `NOT_FOUND` when HIGH-quality data is unavailable. Returned data consists of metadata and URLs to GeoTIFF layers, not a list of every building or a computed circular-area summary. To analyze exactly a circle, download and mask the rasters using the requested center and radius.

Use from Python:

```python
from solar_address import prepare_request

result = prepare_request(
    "1600 Amphitheatre Parkway, Mountain View, CA 94043",
    radius=300,
    unit="feet",
    api_key="YOUR_API_KEY",
)
print(result["solar_parameters"])
```

Run offline tests with `python3 -m unittest -v`. Live calls require your API key.

References: [Geocoding API](https://developers.google.com/maps/documentation/geocoding/guides-v3/requests-geocoding), [Solar dataLayers parameters and limits](https://developers.google.com/maps/documentation/solar/reference/rest/v1/dataLayers/get), [Solar data layers](https://developers.google.com/maps/documentation/solar/data-layers).
