# Get tiger

A `make`-based tool for downloading Census [Tiger Line](http://www.census.gov/geo/maps-data/data/tiger.html) Shapefiles and automatically joining them to data downloaded from the [Census API](http://www.census.gov/data/developers/data-sets.html). This Makefile does is automatically.

## Requirements

* ogr2ogr ([GDAL](http://www.gdal.org)) (v1.10+)
* [jq](https://stedolan.github.io/jq) (v1.5+)

On OS X, install [Homebrew](http://brew.sh) and run: `brew install gdal jq`.

## Install

* Download the repo and put the contents in the folder you would like to fill with GIS data.
* Get a [Census API key](http://api.census.gov/data/key_signup.html) (yes, it's pretty bare-bones).
* Put that key in `key_example.ini`, and rename it `key.ini`.

## Use

Running `make` will produce a list of Census geographies available for download.
```bash
> make
Available data sets:
Download with make DATASET
NATION - United States
...
UNSD - Unified school districts
ZCTA5 - Zip code tabulation areas
```

To download one or more, run with name of the dataset, like so:
````bash
# Download the national and state files and data
make NATION STATE
# Download county data and each state's tract data
make COUNTY TRACT
````

These commands will download files into the `tl_2014` directory. After running, you'll get a list of the files created, e.g.:
```bash
> make NATION STATE
tl_2014/NATION/cb_2014_us_nation_5m.shp tl_2014/STATE/tl_2014_us_state.shp
```

Some commands will download many files. For instance, this will download files for the fifty states, DC and Puerto Rico:
````bash
make PLACE
````

To download only some states and territories, use the `STATE_FIPS` variable:
````bash
# Only New York
make PLACE STATE_FIPS=36

# Only DC, Maryland and Virginia
make PLACE STATE_FIPS="11 24 51"
````

You may find a [list of state fips codes](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standard_state_code) handy.

## What data

A current weakness is that data is downloaded with no data dictionary, and cryptic field names. I've included a data dictionary ([data.json](data.json)) for the default fields.

To download different data, see the [Census API documentation](http://www.census.gov/data/developers/data-sets/acs-survey-5-year-data.html) for a complete list. Make your selection, then run:

````bash
make STATE DATA_FIELDS="GEOID B24124_406E B24124_407E"
````
Note that `GEOID` must be the first field. This example will download employment figures for commercial divers and locksmiths.

You could also add these fields to `key.ini`:
````make
DATA FIELDS = GEOID B24124_406E B24124_407E
````
This will override the defaults in the [`Makefile`](Makefile).

### Vintage

By default, the Makefile downloads 2014 data, the most recent year for which [ACS](https://www.census.gov/programs-surveys/acs/) is available. For older years (or newer years, if it's the future), use the `YEAR` variable:
```bash
make STATE YEAR=2013
make STATE YEAR=2015 
```

### Data series

The default data series is ACS 5-year data or `acs5`. To fetch another data set, use the `SERIES` variable.
```bash
make TRACT SERIES=acs1 
```
(This isn't tested for all data series, but should work.)

### Format

By default, this thing spits out Shapefiles. To get GeoJSON, set `format` to `json`:
````bash
make TRACT format=json
make TRACT format=shp # default
````

### Interesting tidbits

* The Census API appends extra geography fields at the end of a request. For example, 'state', 'county', and 'tract' for a tract file. As part of the processing, these are converted to numbers, which reduces their usefulness. Use the GEOID field for joining.
* The AWATER (water area) and ALAND (land area) fields are given in square meters. `ogr2ogr` has trouble with values more than nine digits long, so these will return errors. The Makefile adds LANDKM and WATERKM fields (the same data in square kilometers) to get around this issue.
* Where available, get-tiger will download the [cartographic boundary](https://www.census.gov/geo/maps-data/data/tiger-cart-boundary.html) files, rather than [Tiger/Line](https://www.census.gov/geo/maps-data/data/tiger-line.html) files. The cartographic files are clipped to the shoreline, Tiger/Line files are not.
* Running tasks with the `--jobs` option (e.g. `make --jobs 3`) to take advantage of a fast connection and/or computer.
