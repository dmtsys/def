<img src="https://github.com/uniqpath/info/blob/master/assets/img/def.png?raw=true" width="300px">

## Motivation

The goal was to develop the simplest possible format for short definition files.

It had to be easily readable and writable by humans and relatively easy readable (the parser part) from various computer languages.

The usage from developer's perspective once converted to `.json` by parser scripts is extremely simple.

Parser scripts for `nodejs` not yet included here but coming soon.

`.json`, `.yaml`, `.toml` and other formats were not suitable for our purposes.

## Examples

### country.def

<pre>
country: <b>Slovenia</b>
  founded: 1991
  timezone: UTC+1
  area: 20,273 sq km

  borderCountries:
    country: Austria
      border: 299 km
    country: Croatia
      border: 600 km
    country: Hungary
      border: 94 km
    country: Italy
      border: 218 km
</pre>

### devices.def

<pre>
device: <b>living-room</b>
  network:
    ip: 192.168.0.30
  search:
    provider: server
  music:
    provider: server
      sambaMount: ~/Media/Music
  beatPings:
    endpoint: http://server.url.com/beat

device: <b>lab</b>
  network:
    ip: 192.168.0.50
  search:
    provider: server
  music:
    provider: server
      sambaMount: ~/Media/Music
  beatPings:
    endpoint: http://server.url.com/beat

device: <b>server</b>
  network:
    ip: 192.168.0.20
    globalIp: 101.101.101.101
  offerWeatherService: true
  search:
    thisProvider:
      catalog: default
  music:
    thisProvider:
      catalog: ~/.dmt/user/data/catalogs/music.catalog.txt
  beatMonitors:
    monitor: living-room
    monitor: lab
</pre>

## Useful concepts:

### Implicit ID:

If a key has something indented under it, thus being a dictionary (hashmap), it will automatically get a special key called `id` which will contain the value specified in the first row, example:

<pre>
country: <b>Slovenia</b>
  founded: 1991
  timezone: UTC+1
</pre>

will be parsed as:

```json
"country" : {
  "id": "Slovenia",
  "founded": "1991",
  "timezone": "UTC+1"
}
```

⚠️ keys with name `id` are not valid, so this is invalid:

<pre>
person:
  id: 1 # illegal key name 'id'
  name: John
</pre>

If you think you need integer `id`s which are entered manually (as is usually the case with `.def` files) then you are probably missing the point and you need a proper database.

### Unique root key per .def file:

This is valid:

```
country: Australia
```
(one)

```
country: Australia
country: Canada
country: Tanzania
```
(many)

While this is not:

```
country: Australia
continent: Europe
```

The point here is enforcing the sane structure which can be leveraged with many simple and powerful features. To sum up:

Each .def file can contain a definition for one thing or multiple instance of this thing, it cannot contain multiple things.

Furthermore there exist only lists of Hashmaps and nothing else. For example, this:

```
country: Australia
country: Canada
country: Tanzania
```

will be parsed as:

```json
[
  {
    "id": "Australia"
  },
  {
    "id": "Canada"
  },
  {
    "id": "Tanzania"
  }
]
```

Then it is very easy to add whatever is needed later, example:

```
country: Australia
  population: somenumber
country: Canada
country: Tanzania
```

which is parsed as:

```json
[
  {
    "id": "Australia",
    "population": "somenumber"
  },
  {
    "id": "Canada"
  },
  {
    "id": "Tanzania"
  }
]
```

Problem can arise here:

```javascript
def.country
```

which will be:

```json
{
  "id": "someid"
}
```

in case of one country and:

```json
[
  {
    "id": "someid"
  },
  {
    "id": "someotherid"
  }
]
```

in case of more than one.

In your parsing code you will have to expect this and use `def.listify()` which just makes an `array of one` in case of single value or an `empty array` in absence of any values. Like this:

```javascript
def.listify(def.country).map(country => ... )
```

This is a small price to pay for making the human users of .def format avoid any nasty surprises and having the simplest format possible. The minimal hassle is chosen to be imposed on the developer instead of end user (as it should always be).

### Multi:

Parsed root element example:

<pre>
devices = {
  device: { name: 'device1', ... }
  <b>multi</b>: [
    { name: 'device1', ... },
    { name: 'device2', ... },
    { name: 'device3', ... },
    ...
    { name: 'deviceN', ... }
  ]
}
</pre>

Each root element is a Hashmap with two keys, first one is the first (or unique if only one) instance of a thing and `multi` accesses a complete list of defined things (or a list of one if unique thing).

