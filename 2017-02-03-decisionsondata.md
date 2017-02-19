---
title: GTFS Is Cool and You Should Use It.
author: Vincent Engelmann
---

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
