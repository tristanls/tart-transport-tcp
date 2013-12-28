# tart-transport-tcp

_Stability: 1 - [Experimental](https://github.com/tristanls/stability-index#stability-1---experimental)_

[![NPM version](https://badge.fury.io/js/tart-transport-tcp.png)](http://npmjs.org/package/tart-transport-tcp)

TCP transport implementation for [Tiny Actor Run-Time in JavaScript](https://github.com/organix/tartjs).

## Contributors

[@dalnefre](https://github.com/dalnefre), [@tristanls](https://github.com/tristanls)

## Overview

An implementation of a TCP transport for [Tiny Actor Run-Time in JavaScript](https://github.com/organix/tartjs).

  * [Usage](#usage)
  * [Tests](#tests)
  * [Documentation](#documentation)
  * [Sources](#sources)

## Usage

To run the below example run:

    npm run readme

```javascript
"use strict";

var net = require('net');
var tart = require('tart');
var transport = require('../index.js');

var sponsor = tart.minimal();

var send = sponsor(transport.sendBeh);

var receivedMessageCount = 0;
var receptionist = sponsor(function (message) {
    console.log('received message:', message);
    receivedMessageCount++;
    if (receivedMessageCount >= 2) {
        close(); // close listening server
    }
});

var serverCapabilities = transport.server(receptionist);
var close = sponsor(serverCapabilities.closeBeh);
var listen = sponsor(serverCapabilities.listenBeh);

var fail = sponsor(function (error) {
    console.dir(error);
});

var listenAck = sponsor(function listenAckBeh(message) {
    console.log('transport listening on tcp://' + message.host + ':' + message.port);
    send({
        address: 'tcp://localhost:7847', 
        content: '{"some":{"json":"content"},"foo":true}',
        fail: fail,
        ok: function () {
            console.log('foo sent');
        }
    });
    send({
        address: 'tcp://localhost:7847', 
        content: '{"some":{"json":"content"},"bar":true}',
        fail: fail,
        ok: function () {
            console.log('bar sent');
        }
    });    
});

listen({host: 'localhost', port: 7847, customer: listenAck});
```

## Tests

    npm test

## Documentation

**Public API**

  * [transport.sendBeh](#transportsendbeh)
  * [transport.server(receptionist)](#transportserverreceptionist)
  * [serverCapabilities.closeBeh](servercapabilitiesclosebeh)
  * [serverCapabilities.listenBeh](servercapabilitieslistenbeh)

### transport.sendBeh

Actor behavior that will attempt to send messages over TCP.

Message format:

  * `address`: _String_ TCP address in URI format scheme, host, port required, and optional but usually necessary fragment. For example: `tcp://localhost:7847/#t5YM5nxnJ/xkPTo...`. 
  * `content`: _String_ JSON content to be sent.
  * `fail`: _Actor_ `function (error) {}` _(Default: undefined)_ Optional actor to report `error` (if any).
  * `ok`: _Actor_ `function () {}` _(Default: undefined)_ Optional actor to report successful send to the destination.

```javascript
var send = sponsor(transport.sendBeh);
send({address: 'tcp://localhost:7847', content: '{"some":{"json":"content"}}'});
```

### transport.server(receptionist)

  * `receptionist`: _Actor_ `function (message) {}` Actor to forward traffic received by this server in {address: <token>, contents: <json>} format.
  * Return: _Object_ An object containing behaviors for listen and close capabilities.
    * `closeBeh`: [serverCapabilities.closeBeh](servercapabilitiesclosebeh)
    * `listenBeh`: [serverCapabilities.listenBeh](servercapabilitieslistenbeh)

Creates an entangled pair of capabilities that will control a single TCP server.

### serverCapabilities.closeBeh

Actor behavior that will close a listening server.

Message is an `ack` _Actor_ `function () {}`, an actor that will be sent an empty message once the server closes.

```javascript
var serverCapabilities = transport.server(receptionist);
var close = sponsor(serverCapabilities.closeBeh);
close(sponsor(function ack() {
    console.log('acked close'); 
});
```

### serverCapabilities.listenBeh

Actor behavior that will create a new listening TCP server.

Message format:

  * `customer`: _Actor_ `function (message) {}` Actor to receive acknowledgment once the server is listening.
  * `host`: _String_ TCP host to listen on.
  * `port`: _Number_ TCP port to listen on.

```javascript
var serverCapabilities = transport.server(receptionist);
var listen = sponsor(serverCapabilities.listenBeh);
listen({
    host: 'localhost',
    port: 7847,
    customer: sponsor(function listenAck(message) {
        console.log('transport listening on tcp://' + message.host + ':' + message.port);
    })
});
```

## Sources

  * [Tiny Actor Run-Time (JavaScript)](https://github.com/organix/tartjs)