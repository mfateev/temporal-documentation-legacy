# Configuring Temporal Server

Temporal Server configuration is found in `development.yaml` 


- [**server**](#server) 
- [**persistence**](#persistence) 
- [**log**](#log) 
- [**clusterMetadata**](#clusterMetadata)
- [**dcRedirectionPolicy**](#dcRedirectionPolicy)
- [**services**](#service) 
- [**kafka**](#kafkaConfig) 
- [**archival**](#archival) 
- [**publicClient**](#publicClient) 
- [**dynamicConfigClient**](#fileBasedClientConfig)
- [**DomainDefaults**](#domainDefaults)

**Note:** Any changing any properties in `development.yaml` file, you must restart the services for changes to take effect.

## server

`server` holds process-wide service-related configuration. See below for a minimal server configuration, with optional paramters are commented out.
```yaml
server:
  ringpop: 
    name: temporal
    //maxJoinDuration: 30s
    //broadcastAddress: "127.0.0.1"
  //pprof:
    //port: 7936

```

### ringpop - *required*

The ringpop section controls the following membership layer parameters:

- `name` - *required* - used to identify other cluster members in a ringpop ring. This must be the same for all nodes.
- `maxJoinDuration` - The amount of time the service will attempt to join the gossip layer before failing.
- `broadcastAddress` - Used as the address that is communicated to remote nodes to connect on. This is generally used when BindOnIP would be the same across several nodes (ie: 0.0.0.0) and for nat traversal scenarios. `net.ParseIP` controls the supported syntax. Note: Only IPV4 is supported.

### pprof

- `port` - If specified, this will initialize pprof upon process start on the listed port.

## persistence
`persistence` holds configuration for the data store / persistence layer. Below is an example minimal specification for a password-secured Cassandra cluster.
```yaml
persistence:
  defaultStore: default
  visibilityStore: visibility
  numHistoryShards: 512
  datastores:
    default:
      cassandra:
        hosts: "127.0.0.1"
        keyspace: "temporal"
        user: "mark"
        password: "password"
    visibility:
      cassandra:
        hosts: "127.0.0.1"
        keyspace: "temporal_visibility"
```

- `defaultStore` - required - The name of the DataStore definition that should be used by the Temporal server
- `visiblityStore` - required - The name of the DataStore definition that should be used by the Temporal visibility server
- `datastores` - required - contains named DataStore definitions to be referenced.
  - Each definition is defined with a heading declaring a name (ie: `default:` and `visibility:` above), which contains a DataStore definition.
  - DataStore definions must be either `cassandra` or `sql`.

A `cassandra` datastore definition can contain the following values:

- `hosts` - *required* - a csv of cassandra endpoints
- `port` - default: 9042 - cassandra port used for connection by gocql client
- `user` - cassandra user used for authentication by gocql client
- `password` - cassandra password used for authentication by gocql client
- `keyspace` - *required* -  the cassandra keyspace
- `datacenter` - the data center filter arg for cassandra
- `maxConns` - the max number of connections to this datastore for a single TLS configuration
-  `tls` - See TLS below

A `sql` datastore definition can contain the following values:

- `user` - user used for authentication
- `password` - password used for authentication
- `pluginName` - *required* - SQL database type
  - *Valid values*: `mysql` or `postgres`
- `databaseName` - *required* - the name of SQL database to connect to
- `connectAddr` - *required* - the remote addr of the database
- `connectProtocol` - *required* - the protocol that goes with the `connectAddr`
  - *Valid values*: `tcp` or `unix`
- `connectAttributes` - a map of key-value attributes to be sent as part of connect data_source_name url
- `maxConns` - the max number of connections to this datastore
- `maxIdleConns` - the max number of idle connections to this datastore
- `maxConnLifetime` - is the maximum time a connection can be alive
- `maxConnLifetime time.Duration `yaml:"maxConnLifetime"`
- `numShards` - number of storage shards to use for tables in a sharded sql database. (*Default:* 1)
- `tls` - See below

`tls` sections can contain:
- `enabled` - `boolean`
- `serverName` - name of the server hosting the datastore
- `certFile` - path to cert file
- `keyFile` - path to key file
- `caFile` - path to the ca file
- `enableHostVerification` - `boolean` - If you want to verify the hostname and server cert (like a wildcard for cass cluster) then you should turn this on. This option is basically the inverse of InSecureSkipVerify. See InSecureSkipVerify in http://golang.org/pkg/crypto/tls/ for more info	

Note: CertPath and KeyPath are optional depending on server config, but both fields must be omitted to avoid using a client certificate


## log
## clusterMetadata

`clusterMetadata` contains all cluster definitions, including those which participate in cross DC.

An example clusterMetadata section:
```yaml
clusterMetadata:
    enableGlobalDomain: false
    failoverVersionIncrement: 10
    masterClusterName: "active"
    currentClusterName: "active"
    clusterInformation:
        active:
            enabled: true
            initialFailoverVersion: 0
            rpcAddress: "127.0.0.1:7233"
    //replicationConsumer:
      //type: kafka
```
- `currentClusterName` - *required* - the name of the current cluster. **Warning**: This value is immutable and will be ignored after 
- `enableGlobalDomain` - *Default:* false
- `replicationConsumerConfig` - determines which method to use to consume replication tasks. The type may be either `kafka` or `rpc`.
- `failoverVersionIncrement` - the increment of each cluster version when failover happens
- `masterClusterName` - the master cluster name, only the master cluster can register / update domain. All clusters can do domain failover.
- `clusterInformation` - contains a map of cluster names to ClusterInformation definitions. ClusterInformation sections consist of:
  - `enabled` - *boolean*
  - `InitialFailoverVersion`
  - `rpcAddress` indicate the remote service address(Host:Port). Host can be DNS name.
	
## dcRedirectionPolicy
## services
`services` contains configuration keyed by service role type. There are four supported service roles:

- `frontend`
- `matching`
- `worker`
- `history`

Below is a minimal example of a `frontend` service definition under `services` 
```yaml
services:
  frontend:
    rpc:
      grpcPort: 8233
      membershipPort: 8933
      bindOnLocalHost: true
    metrics:
      statsd:
        hostPort: "127.0.0.1:8125"
        prefix: "temporal_standby"

```

There are two sections defined under each service heading:

### rpc - *required*
`rpc` contains settings related to the way a service interacts with other services. The following values are supported:

- `grpcPort` is the port  on which gRPC will listen
- `membershipPort` - used by the membership listener (ringpop)
- `bindOnLocalHost` - uses localhost as the listener address
- `bindOnIP` - used to bind service on specific ip (eg. `0.0.0.0`) - check net.ParseIP for supported syntax, only IPv4 is supported, mutually exclusive with `BindOnLocalHost` option.
- `DisableLogging` - disables all logging for rpc
- `logLevel` - the desired log level

**Note**: Port values are currently expected to be consistent among role types across all hosts. 

### metrics
`metrics` contains configuration for the metrics subsystem keyed by provider name. There are three supported providers:

- `statsd`
- `prometheus`
- `m3`

The `statsd` sections supports the following settings:

- `hostPort` - the statsd server host:port
- `prefix` - specific prefix in reporting to statsd
- `flushInterval` - maximum interval for sending packets. (*Default* 1 second)
- `flushBytes` - specifies the maximum udp packet size you wish to send. (*Default* 1432 bytes)

Additionally, metrics supports the following non-provider specific settings:

- `tags` - the set of key-value pairs to be reported
- `prefix` - sets the prefix to all outgoing metrics

## kafka
## archival
## publicClient 
`publicClient` is a required section that contains a single value `hostPort` that is used to specify IPv4 address or dns name and port to reach a frontend client.

Example:
```yaml
publicClient:
  hostPort: "localhost:8933"
```

## dynamicConfigClient
## DomainDefaults