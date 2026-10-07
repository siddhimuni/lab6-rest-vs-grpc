# Lab 6 - REST vs gRPC

Times are in milliseconds per operation.

|  Method | Local | Same-Zone | Different Region |
|---|---|---|---|
| REST add | 3.66 | 4.16 | 343.64 |
| gRPC add | 0.91 | 1.26 | 164.38 |
| REST rawimg | 6.22 | 10.44 | 1375.71 |
| gRPC rawimg | 13.70 | 15.21 | 209.93 |
| REST dotproduct | 3.99 | 4.69 | 390.44 |
| gRPC dotproduct | 1.02 | 1.43 | 145.90 |
| REST jsonimg | 41.29 | 46.29 | 1673.24 |
| gRPC jsonimg | 39.45 | 42.04 | 282.69 |
| PING | 0.046 | 0.443 | 147.81 |

## Observations

gRPC was about 3-4x faster than REST on the small calls (add, dotproduct) locally and in the same zone, thanks to compact binary messages and a reused connection. Network latency dominates once the server is far away: ping to Frankfurt is about 148 ms, and REST add took about 2.3 round trips (344 ms) while gRPC add took about 1.1 (164 ms). This is because REST makes a new TCP connection for every query, paying an extra round trip each time, while gRPC makes one connection and reuses it for all queries. The gap is biggest for the 1.6 MB images across regions (REST 1.4-1.7 s vs gRPC 0.2-0.3 s), since every REST call sends the image over a fresh connection. For large local payloads gRPC had no advantage (rawimg was slower, 13.7 ms vs 6.2 ms) and jsonimg was about 40 ms for both, dominated by base64 encoding rather than the RPC mechanism.