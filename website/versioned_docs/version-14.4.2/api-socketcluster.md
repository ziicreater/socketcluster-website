---
id: version-14.4.2-api-socketcluster
title: API - SocketCluster (Server)
sidebar_label: SocketCluster (server, master)
original_id: api-socketcluster
---

The SocketCluster object represents an entire SocketCluster instance (master process which manages a number of child processes).
This object's constructor accepts an 'options' object which allows you to configure your SC instance.

Note that since SC v2.2.38, the options object can be accessed from any backend process. E.g:
socketCluster.options (in master process - server.js), worker.options (in workerController) and broker.options (in brokerController).

You can also add custom properties to the options object if you want to pass data to various processes. Just make sure that whatever custom data
you provide can be serialized to JSON (so you can't pass functions around - However, you can provide a module name as a string and
require() the module from inside the target process).

### Inherits from:

[EventEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter)

### Properties:

<table>
    <tr>
        <td>options</td>
        <td>Internal options object which defines the behaviour of SC.</td>
    </tr>
</table>

### Events:

<table>
    <tr>
        <td>'fail'</td>
        <td>Any error from any child process or master will cause the 'fail' event to be emitted on your SocketCluster instance (assuming the propagateErrors option is not set to false).</td>
    </tr>
    <tr>
        <td>'warning'</td>
        <td>Triggered by a warning from any child process or master.</td>
    </tr>
    <tr>
        <td>'ready'</td>
        <td>Happens when SocketCluster has booted up and is ready to accept connections.</td>
    </tr>
    <tr>
        <td>'workerStart'</td>
        <td>Emitted whenever a worker is launched. This event's handler can take a workerInfo object as argument. This workerInfo object has an id property (the lead worker will always have id 0), a pid property and a respawn property which indicates whether or not the worker respawned (not the first launch).</td>
    </tr>
    <tr>
        <td>'workerExit'</td>
        <td>Emitted whenever a worker exits. This event's handler can take a workerInfo object as argument. This workerInfo object has an id property (the id of the worker), a pid property, a code property (the exit code) and a signal property (if terminated using a signal).</td>
    </tr>
    <tr>
        <td>'workerMessage'</td>
        <td>
            Emitted when a worker process sends a message to this master process.
            The first parameter passed to the handler is the worker id, the second parameter is the data/object sent by the worker, the third parameter is the respond callback.
            See sendToMaster() method in <a href="/docs/14.4.2/api-scworker">SCWorker API</a> for details on how
            to send a message to master from inside a worker process.
        </td>
    </tr>
    <tr>
        <td>'workerClusterStart'</td>
        <td>Emitted whenever the WorkerCluster is launched (the WorkerCluster handles load balancing between workers). This event's handler can take a workerClusterInfo object as argument. This workerClusterInfo object has a pid property and a childProcess property which is a reference to the WorkerCluster process.</td>
    </tr>
    <tr>
        <td>'workerClusterReady'</td>
        <td>Emitted whenever the WorkerCluster is ready (after all of its child workers have launched). This event's handler can take a workerClusterInfo object as argument. This workerClusterInfo object has a pid property and a childProcess property which is a reference to the WorkerCluster process.</td>
    </tr>
    <tr>
        <td>'workerClusterExit'</td>
        <td>Emitted whenever the WorkerCluster exits. This event's handler can take a workerClusterInfo object as argument. This workerClusterInfo object has a pid property, a code property (the exit code) and a signal property (if terminated using a signal) and a childProcess property which is a reference to the WorkerCluster process.</td>
    </tr>
    <tr>
        <td>'brokerStart'</td>
        <td>Emitted whenever a broker is launched. This event's handler can take a brokerInfo object as argument. This brokerInfo object has an id property (the lead broker will always have id 0), a pid property and a respawn property which indicates whether or not the broker respawned (not the first launch).</td>
    </tr>
    <tr>
        <td>'brokerExit'</td>
        <td>Emitted whenever a broker exits. This event's handler can take a brokerInfo object as argument. This brokerInfo object has an id property (the id of the broker), a pid property, a code property (the exit code) and a signal property (if terminated using a signal).</td>
    </tr>
    <tr>
        <td>'brokerMessage'</td>
        <td>
            Emitted when a broker process sends a message to this master process.
            The first parameter passed to the handler is the broker id, the second parameter is the data/object sent by the broker, the third parameter is the respond callback.
            See sendToMaster() method in <a href="/docs/14.4.2/api-broker">Broker API</a> for details on how
            to send a message to master from inside a broker process.
        </td>
    </tr>
</table>

### Methods:

<table>
    <tr>
        <td>constructor(options)</td>
        <td>
            Creates and launches a SocketCluster instance (with config options).
            The options object can have any of the following properties (sample showing defaults):

```js
{
  // The port on which to listen for outside connections/requests
  port: 8000,

  // Number of worker processes
  workers: 1,

  // Number of broker processes
  brokers: 1,

  // Should be either 'dev' or 'prod' - This affects the shutdown procedure
  // when a 'SIGUSR2' signal is received by master. In 'dev', a SIGUSR2 will
  // trigger an immediate shutdown of workers. In 'prod' workers will
  // be terminated progressively in accordance with processTermTimeout.
  environment: 'dev',

  // Setting this to true will cause the master process to shut down when
  // it receives a SIGUSR2 signal (instead of just the workers).
  // If you're using nodemon, set this to true.
  killMasterOnSignal: false,

  // A unique name for your app (this is used internally for
  // various things such as the directory name in which to store socket
  // file descriptors) - If you don't provide an appName, SC will
  // generate a random one (UUID v4)
  appName: null,

  // This can be the name of an npm module or a path to a Node.js module
  // to use as the WebSocket server engine.
  wsEngine: 'ws',

  // An ID to associate with this specific instance of SC
  // this may be useful if you are running an SC app on multiple
  // hosts - You can access the instanceId from the Broker object
  // (inside brokerController) - If you don't provide an instanceId,
  // SC will generate a random one (UUID v4)
  instanceId: null,

  // A key which various SC processes will use to interact with
  // scBroker broker processes securely, defaults to a 256 bits
  // cryptographically random hex string
  secretKey: null,

  // The key which SC will use to encrypt/decrypt authTokens, defaults
  // to a 256 bits cryptographically random hex string
  // The default JWT algorithm used is 'HS256'.
  // If you want to use RSA or ECDSA, you should provide a authPrivateKey
  // and authPublicKey instead of authKey.
  authKey: null,

  // perMessageDeflate compression. Note that this option is passed directly to the wsEngine's
  // Server object. So if you're using 'ws' as the engine, you can pass an object instead of
  // a boolean. Search for perMessageDeflate here:
  // https://github.com/websockets/ws/blob/master/doc/ws.md#new-websocketserveroptions-callback
  // Note that by default, per-message deflate only kicks in for messages > 1024 bytes.
  perMessageDeflate: false,

  // If using an RSA or ECDSA algorithm to sign the authToken, you will need
  // to provide an authPrivateKey and authPublicKey in PEM format (string or Buffer).
  authPrivateKey: null,
  authPublicKey: null,

  // The default expiry for auth tokens in seconds
  authDefaultExpiry: 86400,

  // The algorithm to use to sign and verify JWT tokens.
  authAlgorithm: 'HS256',

  // Crash workers when an error happens - This is the most sensible default
  crashWorkerOnError: true,

  // Reboot workers when they crash - This is a necessity
  // in production but can be turned off for debugging
  rebootWorkerOnCrash: true,

  // Kill/respawn a worker process if its memory consumption exceeds this
  // threshold (in bytes) - If this is null (default), this behavior
  // will be switched off
  killWorkerMemoryThreshold: null,

  // Can be 'http' or 'https'
  protocol: 'http',

  // This is the same as the object provided to Node.js's https server
  protocolOptions: null,

  // A log level of 3 will log everything, 2 will show notices and errors,
  // 1 will only log errors, 0 will log nothing
  logLevel: 2,

  // In milliseconds, how long a client has to connect to SC before timing out
  connectTimeout: 10000,

  // In milliseconds - If the socket handshake hasn't been completed before
  // this timeout is reached, the new connection attempt will be terminated.
  handshakeTimeout: 10000,

  // In milliseconds, the timeout for calling res(err, data) when
  // your emit() call expects an ACK response from the other side
  // (when callback is provided to emit)
  ackTimeout: 10000,

  // In milliseconds, the timeout for calling res(err, data) when
  // your sendToWorker, sendToBroker or sendToMaster (IPC) call
  // expects an ACK response from the other process
  // (when callback is provided)
  ipcAckTimeout: 10000,

  // In milliseconds - If the socket cannot upgrade transport
  // within this period, it will stop trying
  socketUpgradeTimeout: 1000,

  // Origins which are allowed to connect to the real-time scServer
  origins: '*:*',

  // The maximum number of unique channels which a single
  // socket can subscribe to
  socketChannelLimit: 1000,

  // The interval in milliseconds on which to
  // send a ping to the client to check that
  // it is still alive
  pingInterval: 8000,

  // How many milliseconds to wait without receiving a ping
  // before closing the socket
  pingTimeout: 20000,

  // Maximum amount of milliseconds to wait before force-killing
  // a process after it was passed a 'SIGTERM' or 'SIGUSR2' signal
  processTermTimeout: 10000,

  // Whether or not errors from child processes (workers and brokers)
  // should be passed to the current master process
  propagateErrors: true,

  // Whether or not warnings from child processes (workers and brokers)
  // should be passed to the current master process
  propagateWarnings: true,

  // Whether or not a 'warning' event should be emitted (and logged to console)
  // whenever an action is blocked by a middleware function
  middlewareEmitWarnings: true,

  // Lets you specify a host name to bind to - Defaults to
  // 127.0.0.1 (localhost)
  host: null,

  // The path to a file used to bootstrap worker processes
  workerController: null,

  // The path to a file used to bootstrap broker processes
  brokerController: null,

  // The path to a file used to bootstrap the workerCluster process
  workerClusterController: null,

  // By default, SC will reboot all workers when it receives a 'SIGUSR2' signal -
  // This can be used for updating workers with fresh source code in production
  rebootOnSignal: true,

  // If you run your master process (server.js) as super user, this option
  // lets you downgrade worker and broker processes to run under
  // the specified user (with fewer permissions than master) - You can provide
  // a Linux UID or username
  downgradeToUser: false,

  // The URL path reserved by SocketCluster clients to interact with the server
  path: '/socketcluster',

  // The root directory in which to store your socket files in Linux.
  socketRoot: null,

  // Defaults to "rr", but can be set to "none" - <a href="https://nodejs.org/dist/latest-v5.x/docs/api/cluster.html#cluster_cluster_schedulingpolicy">Click here</a> for details.
  schedulingPolicy: 'rr',

  // Whether or not clients are allowed to publish messages to channels
  allowClientPublish: true,

  // By default, when you send a 'kill -SIGUSR2' signal to the master process,
  // it will reboot all workers, you can turn this behavior off by setting this
  // option to false
  rebootOnSignal: true,

  // This option is passed to the Node.js HTTP server if provided
  tcpSynBacklog: null,

  // SC keeps track of request per minutes internally - This allows you to change
  // how often this gets updated
  workerStatusInterval: 10000,

  // This allows you to batch multiple messages together when passing them across
  // message brokers. This may improve the efficiency of your pub/sub operations.
  // This value is in milliseconds. 5 is generally a safe value to set this to.
  pubSubBatchDuration: null,

  // The default clustering/brokering engine (Node.js module name) which provides the
  // SCWorker.exchange object and manages brokers behind the scenes.
  // You shouldn't need to change this unless you want to build your own
  // process clustering engine (which is difficult to do).
  brokerEngine: 'sc-broker-cluster'
}
```
</td>
</tr>
    <tr>
        <td>sendToWorker(workerId, data, [callback])</td>
        <td>
            Send some data to a specific worker process from master. 
            You can handle this data from inside the worker process by listening for the
            'masterMessage' event. See <a href="/docs/14.4.2/api-scworker">here</a> for more details.
            Since SocketCluster v6.6.0, you can provide an optional callback as the third argument; it should be in the
            form <code>function (err, data) { /* ... */ }</code> - Note that the worker will need to respond
            to the 'masterMessage' event by invoking a <code>respond</code> function; this is an easy way to collect data back from a worker process.
            If the worker does not invoke the <code>respond</code> callback, then this callback will receive an instance of <code>TimeoutError</code> as the first argument.
            The timeout is defined by the <code>ipcAckTimeout</code> option - See the start options above for details.
        </td>
    </tr>
    <tr>
        <td>sendToBroker(brokerId, data, [callback])</td>
        <td>
            <p>
            Send some data to a specific broker process from master. 
            You can handle this data from inside the broker process by listening for the
            'masterMessage' event. See <a href="/docs/14.4.2/api-broker">here</a> for more details.
            Since SocketCluster v6.6.0, you can provide an optional callback as the third argument; it should be in the
            form <code>function (err, data) { /* ... */ }</code> - Note that the broker will need to respond
            to the 'masterMessage' event by invoking a <code>respond</code> function; this is an easy way to collect data back from a broker process.
            If the broker does not invoke the <code>respond</code> callback, then this callback will receive an instance of <code>TimeoutError</code> as the first argument.
            The timeout is defined by the <code>ipcAckTimeout</code> option - See the start options above for details.
            </p>
        </td>
    </tr>
    <tr>
        <td>killWorkers()</td>
        <td>Kills all worker processes managed by SocketCluster. They will respawn by default.</td>
    </tr>
    <tr>
        <td>killBrokers()</td>
        <td>Kills all broker processes managed by SocketCluster. They will respawn by default.</td>
    </tr>
    <tr>
        <td>log(message, [time])</td>
        <td>Log a message internally, time is optional. You generally don't need to call this - It's mostly used internally by SC.</td>
    </tr>
</table>