---
title: GTFS Is Cool and You Should Use It.
author: Vincent Engelmann
---

## Summary

When my wife and I first moved in, we wanted to make sure we found a spot somewhere between my job and her job. At the time, she used her car to get in (suburbia) but I took the bus. If you've ever commuted in New Jersey, I'm sure I don't have to explain the pain! But I am a big proponent of reducing car usage as much as possible.

Fortunately the following year, my wife got a job closer to NYC, so it was high time to find a new apartment. I started looking at apartments through the many realestate sites around. In the process, I found myself constantly copying the address of an apartment in a realestate listing, pasting it into Google, and using the map as a visual guide to see how far the apartment was from a bus stop. (The NYC bus station is 10 blocks farther north than the train station, and I worked in midtown at the time, so this was preferable.)

I thought to myself, there must be a way to automate this.

First I thought, can I run apartment searches across several sites (for rental ranges, etc), scrape the addresses in the results, and do some kind of Google web API call?

Considering some time constraints (and, for that matter, Google API call constraints), I decided on a more pragmatic (although far less awesome) approach.

It turned out that Google (who else?) came up with a standardized schema for transit data, called [GTFS](https://developers.google.com/transit/gtfs/reference). And surprisingly, NJ Transit [publishes data in this format](https://www.njtransit.com/mt/mt_servlet.srv?hdnPageAction=MTDevLoginTo)!

My thinking was, perhaps I can list out all the bus lines and sort by something like distance to the NYC bus station or duration of the ride in (the "route").

The GTFS schema needs to be wrangled. It is dense (completely normalized) and difficult to tease out the table relationships. The documentation is fairly good on Google's site, but *creating* your database schema so you can finally query the GTFS CSV files seemed like a full weekend project. Thankfully, as is many times the case, [someone already wrote a python script](http://cbick.github.io/gtfs_SQL_importer/html/index.html) to convert the GTFS CSVs to a Postgres database. I was not interested in a [whole ORM implementation](https://code.google.com/p/gtfsdb/), because this was more of a one-off analysis.

So after getting the CSVs in a Postgres database, we do some [unwieldy queries](https://gist.github.com/engelmav/4246029) to get a list of bus routes into the Port Authority bus terminal in Manhattan. With some DateTime manipulation in either Perl or Postgres I was able to calculate total durations for each route. The other data point I needed was *where* (as in, *what town*) these routes were departing from.

The key here ended up being the ``stop_lat`` and ``stop_lon`` columns of the ``stops`` table. Fantastic, I thought, we're looking at some proper geocoding. I decided (somewhat arbitrarily) to use [Yahoo's geocoding API](http://developer.yahoo.com/boss/geo/). At the time, it was still completely free but no longer seems to be. It certainly is not expensive for a one-off analysis like this.

The result? There were some 43,000 routes leaving arbitrary locations and arriving at the Port Authority. A moment of silence befell me as I recognized the logistic immensity of NJ Transit.

So I took my report of durations of all trips into Manhattan and looked up the town corresponding to the lattitude and longitude using the Yahoo API. I opened up the resulting CSV, sorted by duration, and saw the towns with the shortest trips at the top.

So we moved to the town with the shortest trip!

## Though process

`stops` will show you the names of stops. For example port authority is id 3511

```sql
select * from stops where stop_name like '%AUTHORITY%'
```

I think the idea behind the below query is that it will show all the stop times of every route where the `stop_id` is the port authority

```sql
select * from stop_times where stop_id = (select stop_id from stops where stop_name like '%AUTHORITY%')
```

And now as a join.

```sql
SELECT
  StopName.stop_name
  ,StopTime.arrival_time
FROM stops StopName

LEFT JOIN stop_times StopTime
  ON StopName.stop_id = StopTime.stop_id
WHERE
  StopName.stop_name like '%AUTHORITY%'
```

But we need to figure out what routes these correspond to.
Turns out there's a routes table. It has route_id, route_short_name, route_type
Is `route_id` listed in the query on StopName above? I don't think so. But there may be a couple of tables linking the two concepts.
Remember, we want:

(a) "What routes go to this stop?
(b) "What stops are on those routes?"

We have trip_id. What does the trips table show us?

There is a `route_id` listed in `trips`. Let's get the SQL just to show the `route_id` and link it to `trips`.

```sql
SELECT
--  *
  StopName.stop_name
  ,StopTime.arrival_time
  ,Trip.route_id
FROM stops StopName

LEFT JOIN stop_times StopTime
  ON StopName.stop_id = StopTime.stop_id
LEFT JOIN trips Trip
  ON StopTime.trip_id = Trip.trip_id 
WHERE
  StopName.stop_name like '%AUTHORITY%'
```

So what is listed in routes that interests us? There is nothing "visible" in route. The `route_long_name or `route_short_name` might have helped up but they are just numerics and are not foreign keys that we can look up. So, this might be a dead end.

http://www.google.com/help/hc/images/transitpartners_1106431_objecttablelarge_en.gif

We might want to start from the trip table since it ultimately has links to everything. We see:

A trip has many stop times, which each have one stop
A trip has many shapes, which shows order of sequence, distance, etc

But, a `route` has many trips...

So maybe we should start from route. Let's try a query.

```sql
SELECT
  *
FROM routes BusRoute

LEFT JOIN trips BusTrip
  ON BusRoute.route_id = BusTrip.route_id
LEFT JOIN stop_times BusStopTime
  ON BusTrip.trip_id = BusStopTime.trip_id
LEFT JOIN stops BusStop
  ON BusStopTime.stop_id = BusStop.stop_id
```

`2147429 rows returned (execution time: 23.400 sec; total time: 23.415 sec)`

Now we must pair this down to find what? A route or a trip that has a destination of Port Authority and _some other_ stops?

(Notes from https://developers.google.com/transit/gtfs/reference#TermDefinitions)

Routes:
"Transit routes. A route is a group of trips that are displayed to riders as a single service. So a route HAS MANY trips. Maybe the 113 is a ROUTE.

So I guess "the 114 is a service"..

And a Trip:
"Trips for each route. A trip is a sequence of two or more stops that occurs at specific time." So that's why a trip HAS MANY stop times (stoptime)


We will get as many items listed as there are routes * trips * stoptimes. And we're looking for total lapsed time between each of those and the ultimate stop, provided our direction (how do we find that?)

```sql
SELECT
--  *
  BusRoute.route_id
  ,bustrip.trip_id
  ,bustrip.direction_id
  ,bustrip.trip_headsign
  ,BusStopTime.arrival_time
  ,BusStopTime.departure_time
  ,busstop.stop_name
FROM routes BusRoute

LEFT JOIN trips BusTrip
  ON BusRoute.route_id = BusTrip.route_id
LEFT JOIN stop_times BusStopTime
  ON BusTrip.trip_id = BusStopTime.trip_id
LEFT JOIN stops BusStop
  ON BusStopTime.stop_id = BusStop.stop_id
LEFT JOIN pabtBuses

order by bustrip.trip_id,BusStopTime.departure_time Asc  
LIMIT 500
```

So how do we find trips that INCLUDE Port Authority at some point?

Trips (and routes) that include new york would be 
Route->trip->stoptime->stop.stop_name

```sql
SELECT
--  *
  BusRoute.route_id
  ,bustrip.trip_id
  ,bustrip.direction_id
  ,bustrip.trip_headsign
  ,BusStopTime.arrival_time
  ,BusStopTime.departure_time
  ,busstop.stop_name
FROM routes BusRoute

LEFT JOIN trips BusTrip
  ON BusRoute.route_id = BusTrip.route_id
LEFT JOIN stop_times BusStopTime
  ON BusTrip.trip_id = BusStopTime.trip_id
LEFT JOIN stops BusStop
  ON BusStopTime.stop_id = BusStop.stop_id

WHERE bustrip.trip_id IN (
SELECT
 StopTime.trip_id
FROM stops StopName

LEFT JOIN stop_times StopTime
  ON StopName.stop_id = StopTime.stop_id
WHERE
  StopName.stop_name like '%AUTHORITY%') 

order by bustrip.trip_id,BusStopTime.departure_time Asc  
LIMIT 500
```

Take the string timestamps and make them actual time data types.
http://www.postgresql.org/docs/8.3/static/functions-formatting.html

[Here's the eventual query and its reporting code](https://github.com/engelmav/gtfs_analysis/blob/master/gtfs_port_auth.pl).