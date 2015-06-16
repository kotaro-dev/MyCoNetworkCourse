# isolate the problem

## 問題の切り分け手順と判断理由

**今回のポイント**

* OSI Reference Model
* TCP/IP Model
* port
* check syslog and error log

## 基本的な調査手順  

*利用しているweb service が急に表示されなくなった場合を想定*

#### [1] ping - start position

- ping success -> ([1-1] connect into the server)

```
(ping success result)
```

- ping error   -> ([2] tracert)

```
(ping error result)
```

#### [1-1] connect into the server with RDP or ssh (ping success)

- connet success -> ([1-1-1] check the service and show the log)

- connet error   -> ([1-1-2] check the used port and stand up to go the server room)

###### [1-1-1] check the service and show the log (connect success)

```
tasklist
service list (net start) (sc query type= service|findstr DISPLAY_NAME)
```

###### [1-1-2] check the used port and stand up to go the server room (connect error)

port check

```
[説明]
```

Walk into the server room!

```
[説明]
```

#### [2] tracert - (ping error)

**tracert**
**ipconfig /all |ifconfig (ip a)**

- inside the gateway      -> ([2-1] client area)
- get the server gateway  -> ([2-3] server area)
- loss between the router -> ([2-2] routing path)

```
(result tracert)
```

###### [2-1] client area

```
[説明]
```

###### [2-2] routing path

```
[説明]
```

###### [2-3] server area

```
[説明]
```

#### [3] ping divide into two groups - backyard

**OSI Reference Model and TCP/IP Model**

```
   [OSI Reference Model]    [TCP/IP Model]

       [7]|Application |    |Application      |  APDU
       [6]|Presentation|    |                 |  PPDU
       [5]|Session     |    |                 |  SPDU
 
       [4]|Transport   |    |Transport        |  TPDU
    -----------------------------------------------
[ping] [3]|Network     |    |Internet         |  packet
 
       [2]|Data link   |    |Network Interface|  Frame
       [1]|Physical    |    |                 |  Bit
```
```
       [7] |browser|                        |Apache2|+---+|php|+---+|mysql|
       [6]     │                               ↑
       [5]     │                               │
               │                               │
       [4]     │                               │
               │                               │
    -----------------------------------------------
[ping] [3]     │   [router]         [router]   │
               │    ↑   │           ↑     │    │
       [2]     │    │   │           │     │    │
       [1]     │    │   │           │     │    │
               ↓    │   ↓           │     ↓    │
          [ethernet]┘   └[fiber    ]┘     └[ethernet]
                         [satellite]
```
```
            [Client]                       [Server]
             ┌───┐┌──┐                    ┌───┬────┬───┐
             │┌─┐││@ ┼┐                   │三三三三三三│ 
           ┌─┴───┴┤三││                   │三三三三三三│ 
           └──────┴──┘│                   └─┼─┴────┴───┘ 
                      │                     │
                    ┌─┼────┐             ┌──┼───┐
                    │::::|:┼─────･･･─────┼::::|:│
                    └──────┘             └──────┘ 
                    [router]             [router]
```
