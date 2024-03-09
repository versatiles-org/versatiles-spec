# VersaTiles Stack Specification v2.1

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
