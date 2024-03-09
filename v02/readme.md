# VersaTiles Stack Specification v2.1

## Foreword

This document provides detailed specifications for the VersaTiles stack, a modular framework for creating, storing, serving and displaying map tiles. The stack is structured into four distinct layers, each dedicated to a specific aspect of tile creation, management and presentation.
The specification uses the terminology "SHALL", "SHOULD" and "MAY" to denote mandatory requirements, strong recommendations and optional practices respectively.

## 1. Stack Layers Overview

- **Generator:** Creates map tiles from geographic data sources such as OpenStreetMap (OSM).
- **Server:** Manages the storage and distribution of map tiles to clients.
- **Network:** Enhances security and optimizes performance through load balancing, TLS and caching.
- **Frontend:** Provides the user interface for map interaction, leveraging libraries like MapLibre GL JS.

### 2. Generator Layer

The Generator creates map tiles from geographic data sources such as OpenStreetMap (OSM).

[^req]: <b id="req-2.1">Requirement 2.1</b> The Generator layer SHALL produce map tiles based on input from geographic data sources, such as OpenStreetMap (OSM).

2.1.2 The Generator SHALL output tiles in formats compliant with the specifications detailed in later sections of this document.

**Requirements:**
- Tiles must be packaged in a [*.versatiles containers](container/readme.md).
- Vector tiles must conform to the [Shortbread schema](https://shortbread-tiles.org/).
- Containers must include detailed metadata compliant with [TileJSON 3.0.0](https://github.com/mapbox/tilejson-spec/tree/master/3.0.0), specifically:
  - `attribution` detailing source data copyrights.
  - `vector_layers` describing the vector tiles' layered composition.

**Recommendations:**
- Use optimal compression techniques to efficiently reduce tile size without compromising data integrity. Recommended methods include:
  - Brotli compression for vector tiles.
  - WebP format for raster tiles to improve loading efficiency and reduce bandwidth.

### 3. Server

2.2.1 The Server layer SHALL manage the storage of generated map tiles.

2.2.2 The Server SHALL distribute map tiles to clients upon request, adhering to the HTTP/HTTPS protocols.

2.2.3 The Server SHOULD implement mechanisms to determine its public URL dynamically to facilitate proper resource referencing.

### 2.3 Network

2.3.1 The Network layer SHALL implement security measures, including, but not limited to, Transport Layer Security (TLS).

2.3.2 The Network layer SHOULD provide load balancing functionalities to distribute incoming traffic across multiple server instances efficiently.

2.3.3 The Network layer MAY implement caching strategies to optimize the performance and scalability of tile delivery.

### 2.4 Frontend

2.4.1 The Frontend layer SHALL provide an interactive interface for map visualization and interaction.

2.4.2 The Frontend SHOULD leverage established libraries such as MapLibre GL JS to ensure compatibility and performance.

2.4.3 The Frontend MAY support additional functionalities, such as layer toggling and dynamic data visualization, to enhance user experience.




---



# VersaTiles Stack Specification v2.0

This document provides detailed specifications for the VersaTiles stack, a modular framework designed for the creation, hosting, and display of map tiles. The stack is structured into four distinct layers, each dedicated to a specific aspect of tile management and presentation.

## Stack Layers Overview

- **Generator:** Creates map tiles from geographic data sources such as OpenStreetMap (OSM).
- **Server:** Manages the storage and distribution of map tiles to clients.
- **Network:** Enhances security and optimizes performance through load balancing, TLS and caching.
- **Frontend:** Provides the user interface for map interaction, leveraging libraries like MapLibre GL JS.

## Generator Layer Specifications

Responsible for tile generation, adhering to defined standards and formats.

**Requirements:**
- Tiles must be packaged in a [*.versatiles containers](container/readme.md).
- Vector tiles must conform to the [Shortbread schema](https://shortbread-tiles.org/).
- Containers must include detailed metadata compliant with [TileJSON 3.0.0](https://github.com/mapbox/tilejson-spec/tree/master/3.0.0), specifically:
  - `attribution` detailing source data copyrights.
  - `vector_layers` describing the vector tiles' layered composition.

**Recommendations:**
- Use optimal compression techniques to efficiently reduce tile size without compromising data integrity. Recommended methods include:
  - Brotli compression for vector tiles.
  - WebP format for raster tiles to improve loading efficiency and reduce bandwidth.

## Server Layer Specifications

The Server layer is responsible for serving map tiles via HTTP.

**Requirements:**
- Must recognize and process [*VersaTiles containers*](container/readme.md).
- Requires knowledge of its public URL for resource referencing.
- Mandatory HTTP service provision.
- Adopts a structured folder hierarchy for organized tile and metadata access:
  - `/tiles/`: Central directory for tile retrieval.
    - `/tiles/sources.json`: Comprehensive index of available tile sources.
    - `/tiles/{name}/{z}/{x}/{y}`: Standardized tile access endpoints.
    - `/tiles/{name}/tiles.json`: Incorporates a legitimate [TileJSON 3.0.0](https://github.com/mapbox/tilejson-spec/tree/master/3.0.0) document.
  - `/assets/`: Storage for additional resources like sprites, glyphs, styles, and MapLibre GL JS files.
    - `/assets/sprites/`
    - `/assets/glyphs/`
      - `/assets/glyphs/{name}`: Font names must only contain letters, numbers and underscore
      - `/assets/glyphs/fonts.json`: Index of available fonts.
    - `/assets/styles/`
    - `/assets/maplibre/maplibre.*`: JavaScript and CSS of newest MapLibre GL JS

**Recommendations:**
- Implementation of Cross-Origin Resource Sharing (CORS) to facilitate resource sharing across different domains.
- Configurability via `config.yaml` for custom server setup, including domain configuration, IP/port listening preferences, operational modes (e.g., development vs. production), tile source definition, and static content management:
	```yaml
	# public URL
	domain: 'https://example.org' # required

	# listen to
	listen_ip: '0.0.0.0' # default: 0.0.0.0
	listen_port: 3000 # default: 8080

	# set to true use only minimal recompression and only if necessary, e.g. for development
	fast: true # default: false

	# set tile sources, required
	tile_sources:
	- { name: 'osm', source: './planet.versatiles' }
	- { name: 'landsat', source: 'https://example.org/landsat.versatiles' }

	# set static content, optional
	static_content:
	- { source: './styles', prefix: 'assets/styles' }
	- { source: './frontend.tar' }
	```

## Network Layer Specifications

The Network layer ensures security and performance.

**Requirements:**
- Efficient request routing to the Server layer.
- Minimal routing of duplicated requests.

**Recommendations:**
- Implementation of Transport Layer Security (TLS) to encrypt data in transit.
- Effective caching strategies to reduce load times and server strain for frequently accessed tiles.

## Frontend Layer Specifications

The Frontend layer is the user interface that renders and interacts with the tiles.

**Requirements:**
- Must ensure compatibility with both vector and raster tiles.
