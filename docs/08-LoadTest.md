## Load test

The load test could be found [here](https://github.com/stefanescualexandrumihai/TechnicalChallenge-loadtester/tree/main).

To run it, open the terminal and run:

```bash
TC_PASS=[USER_PASSWORD] ./loadtest.sh
15:09:00  pods=1  cpu=1m  mem=39Mi
15:09:15  pods=1  cpu=362m  mem=40Mi
15:09:30  pods=4  cpu=1000m  mem=40Mi
15:09:45  pods=8  cpu=759m  mem=40Mi
15:10:01  pods=10  cpu=705m  mem=39Mi
15:10:16  pods=10  cpu=729m  mem=39Mi
15:10:31  pods=10  cpu=798m  mem=39Mi
15:10:46  pods=10  cpu=818m  mem=40Mi

Summary:
  Total:        120.4164 secs
  Slowest:      3.1145 secs
  Fastest:      0.0016 secs
  Average:      0.2743 secs
  Requests/sec: 72.7725
  
  Total data:   255490 bytes
  Size/request: 29 bytes

Response time histogram:
  0.002 [1]     |
  0.313 [6273]  |■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
  0.624 [1810]  |■■■■■■■■■■■■
  0.935 [417]   |■■■
  1.247 [182]   |■
  1.558 [59]    |
  1.869 [15]    |
  2.181 [1]     |
  2.492 [1]     |
  2.803 [1]     |
  3.114 [3]     |


Latency distribution:
  10%% in 0.0686 secs
  25%% in 0.1211 secs
  50%% in 0.2069 secs
  75%% in 0.3367 secs
  90%% in 0.5573 secs
  95%% in 0.7718 secs
  99%% in 1.2211 secs

Details (average, fastest, slowest):
  DNS+dialup:   0.0000 secs, 0.0000 secs, 0.0076 secs
  DNS-lookup:   0.0000 secs, 0.0000 secs, 0.0026 secs
  req write:    0.0000 secs, 0.0000 secs, 0.0027 secs
  resp wait:    0.2702 secs, 0.0015 secs, 3.1062 secs
  resp read:    0.0040 secs, 0.0000 secs, 0.3576 secs

Status code distribution:
  [202] 29 responses
  [409] 8734 responses



loadtest stopped — 300s for scale-down (window 300s)
15:11:01  pods=10  cpu=824m  mem=40Mi
15:11:16  pods=10  cpu=676m  mem=39Mi
15:11:32  pods=10  cpu=295m  mem=39Mi
15:11:47  pods=10  cpu=129m  mem=39Mi
15:12:02  pods=10  cpu=21m  mem=39Mi
15:12:17  pods=10  cpu=1m  mem=39Mi
15:12:32  pods=10  cpu=1m  mem=39Mi
15:12:47  pods=10  cpu=1m  mem=39Mi
15:13:02  pods=10  cpu=1m  mem=39Mi
15:13:17  pods=10  cpu=1m  mem=39Mi
15:13:32  pods=10  cpu=1m  mem=39Mi
15:13:47  pods=10  cpu=1m  mem=39Mi
15:14:02  pods=10  cpu=1m  mem=39Mi
15:14:18  pods=10  cpu=1m  mem=39Mi
15:14:33  pods=10  cpu=1m  mem=39Mi
15:14:48  pods=10  cpu=1m  mem=39Mi
15:15:03  pods=10  cpu=1m  mem=39Mi
15:15:18  pods=10  cpu=1m  mem=39Mi
15:15:33  pods=10  cpu=1m  mem=39Mi
15:15:48  pods=10  cpu=1m  mem=39Mi

poduri 1 → 10   CPU avg 254m/pod (target HPA 50m)
```