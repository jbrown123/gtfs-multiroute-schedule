gtfs-multiroute-schedule
========================

View a consolidated schedule for multiple routes based on GTFS (General Transit Feed Specification) data.


Reference docs
- https://developers.google.com/transit/gtfs/reference
- http://www.gtfs-data-exchange.com/agency/utah-transit-authority/
- http://www.sqlite.org/lang.html

Using SQLite for data storage

Schema
------
```sql
	CREATE TABLE routes (route_id, agency_id, route_short_name, route_long_name, route_desc, route_type, route_url, route_color, route_text_color);
	CREATE TABLE stop_times (trip_id,arrival_time,departure_time,stop_id,stop_sequence,stop_headsign,pickup_type,drop_off_type,shape_dist_traveled);
	CREATE TABLE stops (stop_id,stop_code,stop_name,stop_desc,stop_lat,stop_lon,zone_id,stop_url,location_type,parent_station);
	CREATE TABLE trips (route_id,service_id,trip_id,trip_headsign,direction_id,block_id,shape_id);
	CREATE TABLE calendar (service_id,monday,tuesday,wednesday,thursday,friday,saturday,sunday,start_date,end_date);
	CREATE UNIQUE INDEX routes_route_id ON routes(route_id ASC);
	CREATE INDEX stop_times_stop_id ON stop_times(stop_id ASC);
	CREATE INDEX stop_times_trip_id ON stop_times(trip_id ASC);
	CREATE UNIQUE INDEX stops_stop_id ON stops(stop_id ASC);
	CREATE INDEX trips_route_id ON trips(route_id ASC);
	CREATE UNIQUE INDEX trips_trip_id ON trips(trip_id ASC);
	CREATE UNIQUE INDEX calendar_service_id ON calendar(service_id ASC);
```

Importing data
--------------
```sql
	.separator ","
	.import routes.txt routes
	.import stop_times.txt stop_times
	.import stops.txt stops
	.import trips.txt trips
	.import calendar.txt calendar
	-- make times usable by sqlite (doesn't like single digit hours)
	update stop_times set departure_time = "0"||departure_time where length(departure_time) = 7;
	update stop_times set arrival_time = "0"||arrival_time where length(arrival_time) = 7;
```

Retrieving data
---------------
```sql
.headers on
.mode column

--
-- generic select with all the joins in place
-- st = stop_times, s = stops, t = trips, r = routes, c = calendar
--
select ... from stop_times as st join stops as s on (st.stop_id=s.stop_id) join trips as t on (st.trip_id=t.trip_id) join routes as r on (t.route_id=r.route_id) join calendar as c on (c.service_id=t.service_id) where ...;
```

ToDo
----
