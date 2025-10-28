Great — that means your broker was already formatted for **KRaft**. You should **not** run `kafka-storage.sh format` again. Just point the broker at that data dir and start it.

Here’s exactly what to do.

# 1) Verify the existing cluster ID

Open the file you mentioned and confirm it has a `cluster.id=`:

```
source/kafka-data/metadata/meta.properties
```

You should see something like:

```
version=...
cluster.id=PfVYt8IYQ3-...   <-- keep this
node.id=0                  <-- present on newer formats
```

# 2) Point server config to that data dir

Edit your KRaft server config and ensure it uses **that** log dir. On Kafka 4.0, this file is typically:

```
$KAFKA_HOME/config/kraft/server.properties
```

Make sure these key lines exist (adjust the data path to your actual one):

```
# where your meta.properties already lives
log.dirs=/full/path/to/source/kafka-data

# single-node KRaft roles
process.roles=broker,controller
node.id=0

# listeners (PLAINTEXT for local dev)
listeners=PLAINTEXT://:9092,CONTROLLER://:9093
advertised.listeners=PLAINTEXT://localhost:9092

# controller quorum (self for single-node)
controller.quorum.voters=0@localhost:9093

# (optional) developer-friendly timeouts
auto.create.topics.enable=false
num.partitions=1
```

Notes:

- `log.dirs` must be the **parent** that contains `metadata/` (i.e., where `metadata/meta.properties` is).
    
- `node.id` must match the voter id (here `0`) and (if present) the `node.id` inside `meta.properties`.
    

# 3) Start the broker (no re-format needed)

In one terminal:

```bash
export KAFKA_HOME="$HOME/Downloads/kafka_2.13-4.0.0"   # adjust
"$KAFKA_HOME/bin/kafka-server-start.sh" "$KAFKA_HOME/config/kraft/server.properties"
```

If it starts cleanly you’ll see logs like “KafkaServer id=0 started”.

# 4) Create your topic

New terminal:

```bash
"$KAFKA_HOME/bin/kafka-topics.sh" --create \
  --topic trades \
  --bootstrap-server localhost:9092 \
  --partitions 1 --replication-factor 1

"$KAFKA_HOME/bin/kafka-topics.sh" --list --bootstrap-server localhost:9092
```

# 5) Point your producer at localhost

In your Java producer code:

```java
props.put("bootstrap.servers", "localhost:9092");
```

Run it:

```bash
mvn -q -DskipTests package
java -cp target/kafka-producer-poc-1.0.0.jar com.example.ProducerApp trades
```

# 6) Quick sanity check (consumer shell)

```bash
"$KAFKA_HOME/bin/kafka-console-consumer.sh" \
  --topic trades --from-beginning --bootstrap-server localhost:9092
```

---

## Troubleshooting (common with existing `meta.properties`)

- **Broker won’t start, says “incompatible metadata or cluster id”**  
    Your `server.properties` is pointing to a different `log.dirs` than the one with `meta.properties`. Fix the path or remove stale directories.
    
- **`Invalid configuration for process.roles / controller.quorum.voters`**  
    Make sure these four lines are consistent for single-node:
    
    ```
    process.roles=broker,controller
    node.id=0
    listeners=PLAINTEXT://:9092,CONTROLLER://:9093
    controller.quorum.voters=0@localhost:9093
    ```
    
- **Port already in use (9092/9093)**  
    Change both the listener and advertised listener (and the voter port) to free ports, e.g. 9094/9095:
    
    ```
    listeners=PLAINTEXT://:9094,CONTROLLER://:9095
    advertised.listeners=PLAINTEXT://localhost:9094
    controller.quorum.voters=0@localhost:9095
    ```
    
    Then use `bootstrap.servers=localhost:9094` in the producer.
    
- **`node.id` mismatch**  
    If `meta.properties` contains `node.id=1`, either set `node.id=1` and `controller.quorum.voters=1@localhost:9093`, or re-format a new data dir (not recommended since you already have a cluster id).
    

---

If you paste your exact paths for `$KAFKA_HOME` and `source/kafka-data`, I’ll tailor the `server.properties` block precisely for your machine.