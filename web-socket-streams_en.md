### 1. Market access address
```
wss://push.bitchen.io:9000

```  

### 2. heartbeat

The WebSocket API supports two-way heartbeat, and both the Server and the Client can initiate ping messages and the other party returns pong messages.

WebSocket Server send the heartbeat：

```javascript
 {"ping": 18212558000}
```

WebSocket Client should return：


```javascript
 {"pong": 18212558000}
```


Note: the "pong" inside of the data returned has a value of "ping" in the the received note: after WebSocket Client and WebSocket Server is connected, the WebSocket Server launch a heartbeat every 5 s (the frequency may change) to WebSocket Client, a straight minutes without a heartbeat connection will disconnect

WebSocket Client send the heartbeat：

```javascript
 {"ping": 18212558000}
```

WebSocket Server will return：


```javascript
 {"pong": 18212558000}
```

Note: the "pong" inside of the data returned has a value of "ping" in the the received

### 3. depthUpdate/OrderBook

```
subscription content：depth?symbol={symbol}
```

**Payload:**
```javascript
{
  "e": "depthUpdate", //Event type
  "E": 123456789, // Event time
  "s": "ETH_BTC", // Symbol
  "U": 157, // First update ID in event
  "u": 160, // Final update ID in event
  "af": "1", // http api asksFilter
  "bf": "1", // http api bidsFilter
  "b": [ // Bids to be updated
    {
      "p": "1.2", //price
      "q": "1.2", //quantity
      "a": "1.2", //amount
    }
  ],
  "a": [ // Asks to be updated
    {
      "p": "1.2", //price
      "q": "1.2", //quantity
      "a": "1.2", //amount
    }
  ]
}
```

### 4. miniTickerArr

```
subscription content：miniTickerArr 
```
**Payload:**
```javascript
[{
  ....// Same as ticker payload
}]
```

### 5. ticker
```
subscription content：ticker?symbol={symbol}
```

**Payload:**
```javascript
 {
  "e": "24hrTicker", // Event type
  "E": 123456789, // Event time
  "s": "ETH_BTC", // Symbol
  "p": "0.0015", // Price change
  "P": "250.00", // Price change percent
  "w": "0.0018", // Weighted average price
  "x": "0.0009", // Previous day's close price
  "c": "0.0025", // Current day's close price
  "Q": "10", // Close trade's quantity
  "b": "0.0024", // Best bid price
  "B": "10", // Bid bid quantity
  "a": "0.0026", // Best ask price
  "A": "100", // Best ask quantity
  "o": "0.0010", // Open price
  "h": "0.0025", // High price
  "l": "0.0010", // Low price
  "v": "10000", // Total traded base asset volume
  "q": "18", // Total traded quote asset volume
  "O": 0, // Statistics open time
  "C": 86400000, // Statistics close time
  "F": 0, // First trade ID
  "L": 18150, // Last trade Id
  "n": 18151 // Total number of trades
  "la": "$1.02" //Dollar valuation
}
```


### 6. trade
```
subscription content：trade?symbol={symbol}
```

**Payload:**
```javascript
{

 "e": "trade",     // Event type
 "E": 123456789,   // Event time
 "s": "ETH_BTC",    // Symbol
 "t": 12345,       // Trade ID
 "p": "0.001",     // Price
 "q": "100",       // Quantity
 "b": 88,          // Buyer order Id
 "a": 50,          // Seller order Id
 "T": 123456785,   // Trade time
 "m": true,        // Is the buyer the market maker?
 "M": true         // Ignore
}
```
