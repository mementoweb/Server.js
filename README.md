# Linked Data Fragments Server <img src="http://linkeddatafragments.org/images/logo.svg" width="100" align="right" alt="" />
On today's Web, Linked Data is published in different ways,
which include [data dumps](http://downloads.dbpedia.org/3.9/en/),
[subject pages](http://dbpedia.org/page/Linked_data),
and [results of SPARQL queries](http://dbpedia.org/sparql?default-graph-uri=http%3A%2F%2Fdbpedia.org&query=CONSTRUCT+%7B+%3Fp+a+dbpedia-owl%3AArtist+%7D%0D%0AWHERE+%7B+%3Fp+a+dbpedia-owl%3AArtist+%7D&format=text%2Fturtle).
We call each such part a [**Linked Data Fragment**](http://linkeddatafragments.org/).

The issue with the current Linked Data Fragments
is that they are either so powerful that their servers suffer from low availability rates
([as is the case with SPARQL](http://sw.deri.org/~aidanh/docs/epmonitorISWC.pdf)),
or either don't allow efficient querying.

Instead, this server offers **[Triple Pattern Fragments](http://www.hydra-cg.com/spec/latest/triple-pattern-fragments/)**.
Each Triple Pattern Fragment offers:

- **data** that corresponds to a _triple pattern_
  _([example](http://data.linkeddatafragments.org/dbpedia?subject=&predicate=rdf%3Atype&object=dbpedia-owl%3ARestaurant))_.
- **metadata** that consists of the (approximate) total triple count
  _([example](http://data.linkeddatafragments.org/dbpedia?subject=&predicate=rdf%3Atype&object=))_.
- **controls** that lead to all other fragments of the same dataset
  _([example](http://data.linkeddatafragments.org/dbpedia?subject=&predicate=&object=%22John%22%40en))_.

An example server is available at [data.linkeddatafragments.org](http://data.linkeddatafragments.org/).


## Install the server

This server requires [Node.js](http://nodejs.org/) 0.10 or higher
and is tested on OSX and Linux.
To install, execute:
```bash
$ [sudo] npm install -g ldf-server
```


## Use the server

### Configure the data sources

First, create a configuration file `config.json` similar to `config-example.json`,
in which you detail your data sources.
For example, this configuration uses an [HDT file](http://www.rdfhdt.org/)
and a [SPARQL endpoint](http://www.w3.org/TR/sparql11-protocol/) as sources:
```json
{
  "title": "My Linked Data Fragments server",
  "datasources": {
    "dbpedia": {
      "title": "DBpedia 2014",
      "type": "HdtDatasource",
      "description": "DBpedia 2014 with an HDT back-end",
      "settings": { "file": "data/dbpedia2014.hdt" }
    },
    "dbpedia-sparql": {
      "title": "DBpedia 3.9 (Virtuoso)",
      "type": "SparqlDatasource",
      "description": "DBpedia 3.9 with a Virtuoso back-end",
      "settings": { "endpoint": "http://dbpedia.restdesc.org/", "defaultGraph": "http://dbpedia.org" }
    }
  }
}
```

The following sources are supported out of the box:
- HDT files ([`HdtDatasource`](https://github.com/LinkedDataFragments/Server.js/blob/master/lib/datasources/HdtDatasource.js) with `file` setting)
- N-Triples documents ([`TurtleDatasource`](https://github.com/LinkedDataFragments/Server.js/blob/master/lib/datasources/TurtleDatasource.js) with `url` setting)
- Turtle documents ([`TurtleDatasource`](https://github.com/LinkedDataFragments/Server.js/blob/master/lib/datasources/TurtleDatasource.js) with `url` setting)
- JSON-LD documents ([`JsonLdDatasource`](https://github.com/LinkedDataFragments/Server.js/blob/master/lib/datasources/JsonLdDatasource.js) with `url` setting)
- SPARQL endpoints ([`SparqlDatasource`](https://github.com/LinkedDataFragments/Server.js/blob/master/lib/datasources/SparqlDatasource.js) with `endpoint` and optionally `defaultGraph` settings)

Support for new sources is possible by implementing the [`Datasource`](https://github.com/LinkedDataFragments/Server.js/blob/master/lib/datasources/Datasource.js) interface.

### Memento
The Linked Data Fragments server supports the [Memento Protocol](http://mementoweb.org/about/). If your linked data source evolve over time and has multiple versions, Memento makes it straightforward to access and query across the various versions. For instance, the [Memento DbPedia LDF Server](http://fragments.mementodepot.org/) supports about 10 versions of DbPedia starting from 2007. A Memento Client like [Memento for Chrome](http://bit.ly/memento-for-chrome) can be used to navigate the versions in a browser. The command line utility, cUrl, can also be used to see Memento in action. The following example queries the Memento LDF TimeGate to retrieve a Memento of the English DbPedia page around 15 March 2015.

```
curl -IL -H "Accept-Datetime: Wed, 15 Apr 2015 00:00:00 GMT" http://dbpedia.mementodepot.org/timegate/http://dbpedia.org/page/English

HTTP/1.1 302 Found
Date: Tue, 15 Mar 2016 21:07:08 GMT
Location: http://dbpedia.mementodepot.org/memento/20150415000000/http://dbpedia.org/page/English
Vary: accept-datetime
Link: <http://dbpedia.org/page/English>; rel="original",<http://dbpedia.mementodepot.org/timemap/link/http://dbpedia.org/page/English>; rel="timemap"; type="application/link-format",<http://dbpedia.mementodepot.org/memento/20150415000000/http://dbpedia.org/page/English>; rel="memento"; datetime="Wed, 15 Apr 2015 00:00:00 GMT"

HTTP/1.1 200 OK
Date: Tue, 15 Mar 2016 21:07:08 GMT
Content-Type: text/html
Link: <http://dbpedia.org/page/English>; rel="original", <http://dbpedia.mementodepot.org/memento/20150415000000/http://dbpedia.org/page/English>; rel="memento"; datetime="Wed, 15 Apr 2015 00:00:00 GMT", <http://dbpedia.mementodepot.org/timegate/http://dbpedia.org/page/English>; rel="timegate", <http://dbpedia.mementodepot.org/timemap/link/http://dbpedia.org/page/English>; rel="timemap"
Memento-Datetime: Wed, 15 Apr 2015 00:00:00 GMT
```

#### Memento configuration
To enable Memento support, a new section called `timegates` must be added to the `config.json` file along with information about the versions in the `datasources` section. 

The `timegates` section lists all the versions, called memento, of a data source. The LDF server uses this information to connect a particular data source with its versions, and also to build the appropriate Memento URL for the versions.

For example, the `timegates` section below shows 2 versions of DbPedia:
```json
"timegates": {
  "baseURL": "/timegate/",
  "mementos": {
    "dbpedia": {
      "versions": [
        "dbpedia_2015",
        "dbpedia_2014"
      ]
    }
  }
}
```

For these 2 versions, the `datasources` section of the config looks like:
```json
"datasources": {
  "dbpedia": {
    "title":       "DBpedia 2015",
    "description": "DBpedia 2015 with an HDT back-end",
    "type":        "HdtDatasource",
    "settings":    {
      "file": "/data1/dbpedia/cdata_2015.hdt"
    },
    "timegate": true
  },
  "dbpedia_2015": {
    "title":       "DBpedia v2015",
    "description": "DBpedia v2015 with an HDT back-end",
    "type":        "HdtDatasource",
    "settings":    {
      "file": "/data1/dbpedia/cdata_2015.hdt"
    },
    "memento": {
      "interval": ["2015-04-15T00:00:00Z", "2014-09-14T11:59:59Z"]
    }
  },
  "dbpedia_2014": {
    "title":       "DBpedia v2014",
    "description": "DBpedia v2014 with an HDT back-end",
    "type":        "HdtDatasource",
    "settings":    {
      "file": "/data1/dbpedia/cdata_2014.hdt"
    },
    "memento": {
      "interval": ["2014-09-15T00:00:00Z", "2013-06-15T11:59:59Z"]
    }
  }
```

The first datasource named `dbpedia` is the current version or in Memento terms, the Original. This datasource has a parameter called `timegate: true` which indicates to the LDF server that there are mementos available to this datasource. The LDF server will then use the information provided in the `timegates` section to build the approriate Memento responses.

The datasources `dbpedia_2015` and `dbpedia_2014` contains information about the data source and also information about the time interval this version was active in the `memento` section. The time interval must be in [ISO 8601 format](https://en.wikipedia.org/wiki/ISO_8601). 

### Start the server

After creating a configuration file, execute
```bash
$ ldf-server config.json 5000 4
```
Here, `5000` is the HTTP port on which the server will listen,
and `4` the number of worker processes.

Now visit `http://localhost:5000/` in your browser.

### Reload running server

You can reload the server without any downtime
in order to load a new configuration or version.
<br>
In order to do this, you need the process ID of the server master process.
<br>
One possibility to obtain this are the server logs:
```bash
$ bin/ldf-server config.json
Master 28106 running.
Worker 28107 running on http://localhost:3000/.
```

If you send the server a `SIGHUP` signal:
```bash
$ kill -s SIGHUP 28106
```
it will reload by replacing its workers.

Note that crashed or killed workers are always replaced automatically.

### _(Optional)_ Set up a reverse proxy

A typical Linked Data Fragments server will be exposed
on a public domain or subdomain along with other applications.
Therefore, you need to configure the server to run behind an HTTP reverse proxy.
<br>
To set this up, configure the server's public URL in your server's `config.json`:
```json
{
  "title": "My Linked Data Fragments server",
  "baseURL": "http://data.example.org/",
  "datasources": { … }
}
```
Then configure your reverse proxy to pass requests to your server.
Here's an example for [nginx](http://nginx.org/):
```nginx
server {
  server_name data.example.org;

  location / {
    proxy_pass http://127.0.0.1:3000$request_uri;
    proxy_set_header Host $http_host;
    proxy_pass_header Server;
  }
}
```
Change the value `3000` into the port on which your Linked Data Fragments server runs.

If you would like to proxy the data in a subfolder such as `http://example.org/my/data`,
modify the `baseURL` in your `config.json` to `"http://example.org/my/data"`
and change `location` from `/` to `/my/data` (excluding a trailing slash).

## License
The Linked Data Fragments server is written by [Ruben Verborgh](http://ruben.verborgh.org/).

This code is copyrighted by [iMinds – Ghent University](http://mmlab.be/)
and released under the [MIT license](http://opensource.org/licenses/MIT).
