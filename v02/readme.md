# VersaTiles stack specification v2.0

The stack consists of 4 layers:

- **Generator:** generate tiles
- **Server:** serves tiles
- **Network:** handles TLS, caching, ...
- **Frontend:** provides a frontend, e.g. MapLibre GL JS

## Generator

generates tiles, e.g. from OSM data.

**Required**:
- tiles have to be in a [*.versatiles container](container/readme.md)
- vector tiles have to be in Shortbread schema
- containers must provide valid metadata, including correct:
  - `attribution`
  - `vector_layers` for vector tiles

**Recommended**:
- use best possible compression, e.g.:
  - Brotli compression for vector tiles
  - webp format for raster tiles

## Server

**Required**:
- must read [*.versatiles containers](container/readme.md)
- must know it's public URL
- must serve HTTP
- must serve the following folder structure:
  - `/tiles/` - folder for serving tiles and metadata 
    - `/tiles/sources.json`
    - `/tiles/{name}/{z}/{x}/{y}`
    - `/tiles/{name}/tiles.json` - valid [TileJSON 3.0.0](https://github.com/mapbox/tilejson-spec/tree/master/3.0.0)
  - `/assets/`
    - `/assets/sprites/`
    - `/assets/glyphs/`
      - `/assets/glyphs/{name}` - glyphs, font names must only contain letters, numbers and underscore
      - `/assets/glyphs/fonts.json` - list of available fonts
    - `/assets/styles/`
    - `/assets/maplibre/maplibre.*` - JavaScript and CSS of newest MapLibre GL JS

**Recommended**:
- can handle CORS requests
- should handle the following config.yaml:
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

## Network

**Required**:
- proxy valid requests to the server

**Recommended**:
- provide TLS
- handle caching

## Frontend

**Required**:
- handle vector and images tiles
