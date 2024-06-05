# debugging the crow alarm keypad interface

There's messages of different lengths received from the alarm.

```
7e2a20206778a060187e - 10 bytes
7ec400c03140400084407e - 11 bytes
7e0a007e - 4 bytes
7e280020000000017e - 9 bytes
7e280091000080017e - 9 bytes -- the 9 bytes seems to be the types of messages we want
7ea800000000807e - 8 bytes
```

## 9 bytes messages - 72 length

They all share the same:
- 7e28 - begin (looks like 28 is the message type that we care about. 28 looks like status changes and 48 (hex) is zone changes)
- 7e - end 
- 01 7e (the 01 means status change because it contains the first bit to 1, everything else is 0 - for partial alarm, 81 which means first bit is partial, last bit is status change 10 00 00 01 )
- on the 9 byte message we can check bit 63 set to 1 to mean status change
- byte 4 seems to be the main protagonist
- byte 7 and 8 contain fully/partially armed

```
SA - 7e 28 00 91 00 00 80 01 7e (start arm)
SA - 7e 28 00 55 00 00 80 01 7e (second stage)
SA - 7e 28 00 20 00 00 80 01 7e (fully armed)
D  - 7e 28 00 20 00 00 00 01 7e (disarmed)
NA - 7e 28 00 91 00 00 00 81 7e (night arm start arm)
NA - 7e 28 00 55 00 00 00 81 7e (NA second stage)
NA - 7e 28 00 a8 00 00 00 81 7e (NA armed)
T  - 7e 28 00 d9 00 00 80 01 7e
```

```
Trigger d9:
1 1 0 1  1 0 0 1

But a chime is... not correct.
1 1 1 1  1 0 0 1

disarmed
4th byte: 20
0 0 1 0  0 0 0 0

fully arm versus partial arm:
bit 48 = 1 && bit 56 = 0 fully armed
bit 48 = 0 && bit 56 = 1 partially armed
bit 48 = 0 && bit 56 = 0 no arming

Start arm:
4th byte: 91
1 0 0 1  0 0 0 1
 
second stage:
4th byte: 55
0 1 0 1  0 1 0 1

armed:
4th byte: 20
0 0 1 0  0 0 0 0

BUT night arm is: a8 and not 20
1 0 1 0  1 0 0 0
bit 28: it's 1 and not 0 like fully armed.
```

### dump of the 9 bytes messages when doing specific actions

```
Arm from remote:
7e280091000080017e

second stage from arm from remote:
7e280055000080017e

finished arming total from remote:
7e280020000080017e

Disarmed from remote:
7e280020000000017e
7e280020000000017e

Canceled disarm from remote:
7e280020000000017e - bit 63 is 1 from "01" in hex - that's status change.
7e480000000000007e - the bit 63 (last 0, >00<7e) if 0 means zone report, not status change

Arm from keypad:
7e280091000080017e

second stage arming from keypad:
7e280055000080017e

finished arming total from keypad:
7e280020000080017e
(arming from keypad or from remote is the same status.)


partial arm from remote:
7e280091000000817e

second stage arming from remote:
7e280055000000817e

finished arming partial from remote:
7e2800a8000000817e

disarmed from remote:
7e280020000000017e
```