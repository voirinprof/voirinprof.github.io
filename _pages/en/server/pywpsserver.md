---
title: "Introduction to PyWPS"
permalink: /en/pywps/
lang: en
date: 2025-06-11T03:02:20+00:00
toc: true
toc_sticky: true
toc_label: "On this page"
typenav: gisserver
---

## Introduction to PyWPS

### What is PyWPS?

**PyWPS** (Python Web Processing Service) is a free and open-source implementation of the **OGC WPS** (Web Processing Service) standard, which allows spatial processing tasks (Python scripts, geospatial tools, models) to be published as **web services**.

---

### How it works

1. A **client** (e.g., QGIS, browser) sends a WPS request (`GetCapabilities` / `Execute`).
2. **PyWPS** receives the HTTP request and executes a defined Python process.
3. The **result** (value, file, image, GeoJSON, etc.) is returned to the client.

---

### Project structure

A typical PyWPS project looks like this:

```

my-wps/
├── wps_raster_analyser.py # the process
├── pywps.cfg              # WPS service configuration
└── wps.py                 # service entry point

````

---

### Simple process example: `GetRasterProcess`

#### `wps_raster_analyser.py`

Here’s an example that applies a filter to an image retrieved from a WMS server. The advantage of WMS is that data can be requested via web services.

The example below illustrates a Sobel or Prewitt filter applied to an image from a WMS service.

```python
from pywps import Process, LiteralInput, ComplexOutput
from pywps.inout.formats import Format 
from pywps.app.Common import Metadata
import requests
from PIL import Image
from io import BytesIO
import os
import numpy as np

from PIL import ImageFilter

# sobel filter x and y coefficients
sobel_x_coeffs = [
    -1, 0, 1,
    -2, 0, 2,
    -1, 0, 1
]
sobel_y_coeffs = [
    -1, -2, -1,
    0,  0,  0,
    1,  2,  1
]

# prewitt filter x and y coefficients
prewitt_x_coeffs = [
    -1, 0, 1,
    -1, 0, 1,
    -1, 0, 1
]
prewitt_y_coeffs = [
    -1, -1, -1,
    0,  0,  0,
    1,  1,  1
]

class GetRasterProcess(Process):
    """Process to get a raster image from a WMS layer and apply a filter."""
    def __init__(self):
        """Initialize the process with inputs and outputs."""
        inputs = [
            # Input for WMS layer, latitude, longitude, and method
            LiteralInput('wms', 'WMS Layer', data_type='string'),
            LiteralInput('lat', 'Latitude', data_type='float'),
            LiteralInput('lon', 'Longitude', data_type='float'),
            LiteralInput('method', 'Method', data_type='string', default='sobel_x', allowed_values=['sobel_x', 'sobel_y', 'prewitt_x', 'prewitt_y']),
        ]
        # Output for the raster image
        outputs = [
            ComplexOutput('image', 'WMS Result Image',
                          as_reference=True,
                          supported_formats=[Format('image/png')])
        ]
        
        super().__init__(
            self._handler,
            identifier='get_raster_process',
            title='Get Raster Image',
            abstract='Get a 512x512 raster image centered at the given lat/lon.',
            metadata=[Metadata('Raster Image')],
            inputs=inputs,
            outputs=outputs
        )

    # Handler function to process the request
    def _handler(self, request, response):
        lat = request.inputs['lat'][0].data
        lon = request.inputs['lon'][0].data
        wms_layer = request.inputs['wms'][0].data
        method = request.inputs.get('method', ['slic'])[0].data
        
        output_type = request.outputs['image']['mimetype']
        

        # Geoserver Config
        wms_url = "http://geoserver:8080/geoserver/wms"
        layer = wms_layer 
        width, height = 512, 512
        bbox_size_deg = 0.0005  # taille du carré autour du point (à adapter selon l'échelle)

        # Estimate bbox around the point
        minx = lon - bbox_size_deg / 2
        maxx = lon + bbox_size_deg / 2
        miny = lat - bbox_size_deg / 2
        maxy = lat + bbox_size_deg / 2
        bbox = f"{minx},{miny},{maxx},{maxy}"

        # Get the WMS image
        params = {
            "service": "WMS",
            "version": "1.1.1",
            "request": "GetMap",
            "layers": layer,
            "bbox": bbox,
            "width": width,
            "height": height,
            "srs": "EPSG:4326",
            "format": "image/png"
        }
        
        # send the request to the WMS server
        r = requests.get(wms_url, params=params)
        r.raise_for_status()

        # Save the image to a BytesIO object
        img = Image.open(BytesIO(r.content))        
        
        # according to the method, apply the corresponding filter
        if method == 'sobel_x':
            # apply the Sobel filter in X direction
            kernel = np.array(sobel_x_coeffs).reshape((3, 3))
        elif method == 'sobel_y':
            # apply the Sobel filter in Y direction
            kernel = np.array(sobel_y_coeffs).reshape((3, 3))
        elif method == 'prewitt_x':
            # apply the Prewitt filter in X direction
            kernel = np.array(prewitt_x_coeffs).reshape((3, 3))
        elif method == 'prewitt_y':
            # apply the Prewitt filter in Y direction
            kernel = np.array(prewitt_y_coeffs).reshape((3, 3))
        else:
            raise ValueError("Invalid method specified. Use 'sobel_x', 'sobel_y', 'prewitt_x', or 'prewitt_y'.")

        # Apply the filter
        # Convert the image to grayscale if it is not already
        img = img.convert('L')
        # Apply the kernel filter
        img = img.filter(ImageFilter.Kernel((3, 3), kernel.flatten(), scale=1, offset=0))
        
        
        # keep the result in the /app/images directory
        os.makedirs("/app/images", exist_ok=True)

        # Save the image to a file
        image_path = f"/app/images/tile_{method}_{lat}_{lon}.png"
        img.save(image_path)

        # response type
        response.outputs['image'].output_format = Format(output_type)
        response.outputs['image'].file = image_path
        return response

```

---

### Running the service

#### `wps.py`

```python
from pywps import Service
from wps_raster_analyser import GetRasterProcess

processes = [GetRasterProcess()]
application = Service(processes, config_file='pywps.cfg')
```

Then run with:

```bash
gunicorn wps:application
```

---

### Configuration: `pywps.cfg`

```ini
[server]
url = http://localhost:5000/wps
maxprocesses = 10
processes = 5
outputpath = outputs
logfile = pywps.log
loglevel = DEBUG
```

---

### Docker integration

You can use `docker-compose` to deploy the service:

```
pywps:
    build:
      context: ./pywps
    ports:
      - "5001:5000"
    volumes:
      - ./pywps:/app
    
```

If you want to use advanced libraries (like rasterio, gdal...), it's better to define a Dockerfile and a `requirements.txt` file.

**Dockerfile:**

```
FROM python:3.11-slim

RUN apt-get update && apt-get install -y \
    libgdal-dev \
    gdal-bin \
    libexpat1 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["gunicorn", "wps:application", "--bind", "0.0.0.0:5000"]
```

**requirements.txt**

```
pywps
gunicorn
requests
pillow
numpy
rasterio
```

---

### Testing with `curl`

```bash
curl "http://localhost:5000/wps?service=WPS&request=GetCapabilities"
```

---

### Testing in QGIS

1. Go to **Processing > Toolbox > Add WPS Server**.
2. Add the URL: `http://localhost:5000/wps`.
3. The `hello` process should appear and be ready to execute.

---

### Execute the process

```bash
curl --location 'http://localhost:5000/wps/?service=WPS' \
--header 'Content-Type: text/xml' \
--data '<wps:Execute
            xmlns:wps="http://www.opengis.net/wps/1.0.0"
            xmlns:ows="http://www.opengis.net/ows/1.1"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.opengis.net/wps/1.0.0
                                http://schemas.opengis.net/wps/1.0.0/wpsExecute_request.xsd"
            service="WPS"
            version="1.0.0"
            storeExecuteResponse="false"
            status="false">
    <ows:Identifier>get_raster_process</ows:Identifier>
    <wps:DataInputs>
        <wps:Input>
            <ows:Identifier>wms</ows:Identifier>
            <wps:Data>
                <wps:LiteralData>geoimage:lon_bing_mosaic_tiled</wps:LiteralData>
            </wps:Data>
        </wps:Input>
        <wps:Input>
            <ows:Identifier>method</ows:Identifier>
            <wps:Data>
                <wps:LiteralData>sobel_y</wps:LiteralData>
            </wps:Data>
        </wps:Input>
        <wps:Input>
            <ows:Identifier>lat</ows:Identifier>
            <wps:Data>
                <wps:LiteralData>45.55425883140662</wps:LiteralData>
            </wps:Data>
        </wps:Input>
        <wps:Input>
            <ows:Identifier>lon</ows:Identifier>
            <wps:Data>
                <wps:LiteralData>-73.49350601434709</wps:LiteralData>
            </wps:Data>
        </wps:Input>        
    </wps:DataInputs>
    <wps:ResponseForm>
        <wps:RawDataOutput mimeType="image/png">
            <ows:Identifier>image</ows:Identifier>
        </wps:RawDataOutput>
    </wps:ResponseForm>
</wps:Execute>'
```

The result will be a PNG image.

---

### 📚 Resources

* Official documentation: [https://pywps.readthedocs.io/](https://pywps.readthedocs.io/)
* OGC WPS standard: [https://www.ogc.org/standards/wps](https://www.ogc.org/standards/wps)