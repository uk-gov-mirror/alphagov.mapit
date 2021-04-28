# MapIt

This is a fork of [the MapIt repo](https://github.com/mysociety/mapit). MapIt is a Python/Django application (backed by PostgreSQL), that provides RESTful API for looking up postcodes, council boundaries, etc.

**Avoid adding features, changing behaviours, or touching the code of MapIt itself**. The only [differences against the original repo](https://github.com/mysociety/mapit/compare/master...alphagov:master) should only be to help integrate it into the GOV.UK stack. So far this includes:

 - [Pinning to specific versions of Python dependencies](https://github.com/alphagov/mapit/pull/1), for reliability
 - [Adding a Procfile](https://github.com/alphagov/mapit/pull/2) and [unicornherder](https://github.com/alphagov/mapit/pull/12), to standardise deployment with other apps
 - [A GOV.UK specific README](https://github.com/alphagov/mapit/pull/3), to provide context for GOV.UK developers
 - [Importing a DB from S3](import-db-from-s3.sh), to simplify bringing new servers online with the same data

If we need to change the code itself, we should communicate with mysociety to try and push the work back upstream so everyone benefits.

## Nomenclature

- **Area**: a set of points that define an area that represents a geopolitical boundary of some sort.  A country, a parliamentary constituency, a national park, a civil parish, etc.
- **Postcode**: an identifier for a small geographical area usually derived from postal data.  Postcodes don't have boundaries, instead they have a lat/lng point which is the centroid of their area.  The **Area**s for that **Postcode** are looked up by doing a geo search for those that contain the point.
- **Generation**: because areas change over time we add **Area**s to generations.  If an area is in one generation but not another it is likely because legislation was enacted between those generations that meant it ceased to exist (often it is replaced by another similar but not exactly the same Area).  Unless specified in the request mapit only searches in the current generation.
- **Type**: to identify what an area is it has a type.  For example European parliamentary constituencies are type 'EUR' compared to Westminister parliamentary constituencies of type 'WMC'.
- **CodeType**: Areas have codes to allow them to be identified, issued by the ONS.  Prior to 2011 these were SNAC codes (stored as type 'ons') (see: http://www.ons.gov.uk/ons/guide-method/geography/products/names--codes-and-look-ups/standard-names-and-codes/index.html), since 2011 these are GSS codes (stored as type 'gss') (see: http://neighbourhood.statistics.gov.uk/dissemination/Info.do?m=0&s=1460381709047&enc=1&page=nessgeography/new-geography-codes-and-naming-policy-from-1-january-2011.htm&nsjs=true&nsck=false&nssvg=false&nswid=1212)
- **ONSPD**: The Office for National Statistics Postcode Database - one of the open data sources we use to import data. We get UK postcodes from this.
- **Boundary Line**: The Ordnance Survey boundary line data - one of the open data sources we use to import data.  We get GB Areas from this.
- **OSNI**: The Ordnance Survey of Northern Ireland - one of the providers of the open data sources we use to import data.  We get NI Areas from them.
- **ONS**: Office for National Statistics
- **GSS**: Government Statistical Service, new ONS codes - GSS codes start with a letter, and are 9 characters long e.g. `S14000051`
- **SNAC**: old ONS codes - ONS codes consist of a number and/or character combination that could be
 2, 4, 6, 8, or 10 characters long, and depending on the length, may only be numbers
- **SRID**: spatial reference identifier

## Technical documentation

You can use the [GOV.UK Docker environment](https://github.com/alphagov/govuk-docker) to run the application and its tests with all the necessary dependencies. Follow [the usage instructions](https://github.com/alphagov/govuk-docker#usage) to get started.

**Use GOV.UK Docker to run any commands that follow.**

### Running the application

Run `make mapit` in the `govuk-docker` repo - this will build the image and install all the dependencies.

Start your Docker container by running :

    $ govuk-docker up mapit-app

Check you are able to access `mapit.dev.gov.uk` - it is expected that the
frontend looks somewhat "broken", that's okay - we only need to worry about
the database for importing data.

To run any other management commands (`.venv/bin/python ./manage.py ...`), you
will need to be in a bash shell:

    $ docker exec -it <container name> /bin/bash

To run management commands in other environments, you'll need the `GOVUK_ENV` environment variable set.

### Running the test suite

`GOVUK_ENV=development .venv/bin/python ./manage.py test mapit mapit_gb`

Include any other edge cases, e.g parallel test runner in Whitehall

### Further documentation

- [Original README](README.rst)
- [Importing data](docs/importing-data.md)
- [Testing a server with an updated Mapit database](docs/testing-server.md)
- [Example API output](docs/api.md)

## Licence

[GNU Affero GPL](LICENSE.txt)
