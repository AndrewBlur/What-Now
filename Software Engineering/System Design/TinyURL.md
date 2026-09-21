#17/09/2026
built a normal version of tinyurl replica -> shortening URLs to at most 10 chars, 
- used base62_encode method to achieve this
- deployed in docker 
- postgres for db , and fastapi for http server
- tried to simulate 10000 user request , was able to achieve 223/s throughput
- worst response took 25 mins 
- 95% of response took 4 mins 

#18/09/2026
Phase 1
- added basic validations for URLs from pydantic which will throw 422
- added proper error handling
- added per request -> db session management -> created a get_db style 
- changed the db layer to async  -> changes the db driver to support async
- added healthcheck for db start ,   

#21/09/2026
Phase 2
- Added redis for cache-aside pattern -> sped up the reads
- Added neccessary file changes 
Phase 3
- Added Nginx for load balancing 
- as the apps itself doesnt hold any state , the final states are in the db and cache so 
  we can replicate the apps and configure the nginx to balance the request loads
- so manually wrote the replication in docker-compose.yml and ngnix.conf files 
Phase 4
- Graceful shutdown 




What I learned:

core logic of the app -> base62 encoding 
	- encoding the reccuring id of the record in db with base62 format 
	- which in return can encode upto 62^10 which is really huge in just 10 characters 
	- so a tinyurl

`locust` for load testing

adding `healthchecks`

`async db connections`, per request db connections using fastapi's dependency injection

Redis -> is a in memory cache to speed up reads with cache aside patterns , there are more patterns to speed up writes as well 

`redis - library in python offers these functions`
	- client : Redis.from_url(os.getenv("REDIS_URL","redis://localhost:6379/0"),decode_responses=True,)
	- with this client we can store using client.set and retrieve from cache using client.get
	- .set accepts a key, value , ttl 
	- .get accepts just the key
- easy to setup with docker, just pull the image of choice and configure the envs

`Nginx` -> is a reverse proxy which in our app acts a load balancer between replicas of the app itself , we can do replicas because the app itself doesnt store any state all the state user wants exists in a shared db and cache which all app points to

