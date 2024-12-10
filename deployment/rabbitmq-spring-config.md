# Spring Boot RabbitMQ Configuration Explained

## Connection Configuration

```yaml
internal-consumer:
  connection:
    url: "10.80.4.177"
    port: "5672"
    user: "odin"
    password: "helloworld"
    vhost: "odin"
    heartbeat: 30
    addresses:
      - "10.80.4.177"
      - "10.80.4.178"
      - "10.80.4.179"
```

### Connection Details Explained

```mermaid
graph TB
    subgraph Cluster_Setup
        N1[Node 1<br>10.80.4.177]
        N2[Node 2<br>10.80.4.178]
        N3[Node 3<br>10.80.4.179]
    end
    
    Client-->N1
    Client-->N2
    Client-->N3
    
    style N1 fill:#f9f,stroke:#333
    style N2 fill:#f9f,stroke:#333
    style N3 fill:#f9f,stroke:#333
```

1. **Cluster Configuration**
   - Three RabbitMQ nodes in cluster
   - Client can connect to any node
   - Automatic failover support

2. **Connection Parameters**
   - `url`: Primary node address
   - `port`: AMQP default port (5672)
   - `vhost`: "odin" virtual host for isolation
   - `heartbeat`: 30 seconds keepalive
   - `addresses`: List of all cluster nodes

## Channel Configuration

```yaml
channel:
  prefetch_count: 100
  prefetch_size: 0
  global: false
```

### Channel QoS Settings

```mermaid
graph LR
    subgraph Channel_QoS
        PC[Prefetch Count: 100]
        PS[Prefetch Size: 0]
        G[Global: false]
    end
    
    PC --> Consumer
    PS --> Consumer
    G --> Consumer
```

1. **Prefetch Settings**
   - `prefetch_count: 100`: Maximum unacknowledged messages
   - `prefetch_size: 0`: No byte limit for messages
   - `global: false`: Settings apply per consumer

2. **Impact**
   ```mermaid
   sequenceDiagram
       participant Q as Queue
       participant C as Consumer
       
       Note over Q,C: Max 100 unacked messages
       Q->>C: Message 1
       Q->>C: Message 2
       Q->>C: ... (up to 100)
       C-->>Q: Ack Message 1
       Q->>C: Message 101
   ```

## Queue Configuration

```yaml
queues:
  msn05:
    name: "ixu.i.q.msn05"
    cluster: 1
  funny-holiday:
    name: "ixu.i.q.funny-holiday"
    cluster: 1
```

### Queue Structure

```mermaid
graph TB
    subgraph Queues
        Q1[ixu.i.q.msn05]
        Q2[ixu.i.q.funny-holiday]
    end
    
    C[Consumer] --> Q1
    C --> Q2
    
    style Q1 fill:#f96,stroke:#333
    style Q2 fill:#f96,stroke:#333
```

1. **Queue Naming Convention**
   - Prefix: `ixu.i.q`
   - Names: `msn05` and `funny-holiday`
   - Structured naming for organization

2. **Cluster Setting**
   - `cluster: 1`: Queue replication factor
   - Ensures high availability

## Performance Implications

### 1. Connection Level
- Heartbeat every 30 seconds
- Cluster-aware configuration
- Automatic failover support

### 2. Channel Level
```mermaid
graph TB
    subgraph Message_Flow
        Q[Queue]
        C[Consumer]
        B[Buffer: 100 msgs]
    end
    
    Q -->|Deliver| B
    B -->|Process| C
```

- 100 messages in-flight limit
- Per-consumer throttling
- No size restrictions

### 3. Queue Level
- High availability (cluster: 1)
- Organized naming structure
- Multiple queue support

## Best Practices

1. **Connection Management**
   - Use cluster addresses for failover
   - Maintain reasonable heartbeat
   - Proper virtual host isolation

2. **Channel Optimization**
   ```java
   // Spring AMQP example
   @Bean
   public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory() {
       SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
       factory.setPrefetchCount(100);
       factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
       return factory;
   }
   ```

3. **Queue Handling**
   - Use meaningful queue names
   - Set appropriate HA policies
   - Monitor queue depths

## Monitoring Recommendations

1. **Connection Monitoring**
   ```bash
   # Check cluster status
   rabbitmqctl cluster_status
   
   # Monitor connections
   rabbitmqctl list_connections
   ```

2. **Channel Monitoring**
   ```bash
   # Check consumer counts
   rabbitmqctl list_queues name consumers messages
   
   # Monitor channel status
   rabbitmqctl list_channels
   ```

3. **Queue Monitoring**
   ```bash
   # Check queue status
   rabbitmqctl list_queues name messages_ready messages_unacknowledged
   ```
