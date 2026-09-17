#17/09/2026
built a normal version of tinyurl replica -> shortening URLs to at most 10 chars, 
- used base62_encode method to achieve this
- deployed in docker 
- postgres for db , and fastapi for http server
- tried to simulate 10000 user request , was able to achieve 223/s throughput
- worst response took 25 mins 
- 95% of response took 4 mins 

