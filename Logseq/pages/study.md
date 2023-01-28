- PDF upload
- /upload
  type:: [[test]]
- {{query (property type test)}}
  query-table:: true
  query-sort-by:: block
  query-sort-desc:: true
-
- NOW this is a test
  :LOGBOOK:
  CLOCK: [2022-12-17 Sat 20:00:24]--[2022-12-17 Sat 20:00:25] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:00:25]--[2022-12-17 Sat 20:00:26] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:00:26]--[2022-12-17 Sat 20:00:27] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:00:55]--[2022-12-17 Sat 20:00:56] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:01:21]--[2022-12-17 Sat 20:01:22] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:01:22]--[2022-12-17 Sat 20:01:23] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:01:23]--[2022-12-17 Sat 20:16:19] =>  00:14:56
  CLOCK: [2022-12-17 Sat 20:16:19]--[2022-12-17 Sat 20:16:20] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:16:20]--[2022-12-17 Sat 20:16:21] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:16:22]
  :END:
- LATER tests
  :LOGBOOK:
  CLOCK: [2022-12-17 Sat 20:00:45]--[2022-12-17 Sat 20:00:46] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:00:46]--[2022-12-17 Sat 20:00:47] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:00:51]--[2022-12-17 Sat 20:00:52] =>  00:00:01
  CLOCK: [2022-12-17 Sat 20:00:52]--[2022-12-17 Sat 20:00:52] =>  00:00:00
  CLOCK: [2022-12-17 Sat 20:00:53]--[2022-12-17 Sat 20:00:53] =>  00:00:00
  CLOCK: [2022-12-17 Sat 20:00:54]--[2022-12-17 Sat 20:00:54] =>  00:00:00
  CLOCK: [2022-12-17 Sat 20:16:04]--[2022-12-17 Sat 20:16:07] =>  00:00:03
  CLOCK: [2022-12-17 Sat 20:16:11]--[2022-12-17 Sat 20:16:14] =>  00:00:03
  :END:
- TODO aaa
  :LOGBOOK:
  CLOCK: [2022-12-17 Sat 20:16:15]--[2022-12-17 Sat 20:16:17] =>  00:00:02
  :END:
- {{query (and (task todo now later) (sort-by clock-time desc))}}
  query-sort-by:: block
  query-table:: true
  query-sort-desc:: false
- this is a test
  priority:: a
-
- {{query (priority a)}}
  query-table:: true
-
-