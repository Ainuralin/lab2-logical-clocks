# Lab 2 — Logical Clocks and Replication Consistency (3 Nodes)

This repository contains the starter code for **Lab 2**: a 3-node replicated key–value store with **Lamport logical clocks** and **last-writer-wins (LWW) conflict resolution**.  
The lab demonstrates causality, concurrent updates, message delays, and temporary node outages.

---

## Files

- `node.py` — Node server (HTTP JSON API), Lamport clock, replication, LWW conflict resolution  
- `client.py` — CLI client to send `PUT`, `GET`, and `STATUS` requests  
- `report.md` — Lab report (optional PDF version can be added)  

---

## EC2 Setup / Ports

Each node should be deployed on a separate EC2 instance (Ubuntu 22.04).  
Open the following ports in the security group for peer-to-peer communication:

| Node | Port | Private IP example |
|------|------|------------------|
| A    | 8000 | 172.31.17.9      |
| B    | 8001 | 172.31.24.36     |
| C    | 8002 | 172.31.17.55     |

---

## Running Nodes on EC2

Replace `<IP-X>` with the **private IP of the peer node**.

### Node A
```bash
python3 node.py --id A --port 8000 --peers http://<IP-B>:8001,http://<IP-C>:8002
```

### B
```bash
python3 node.py --id B --port 8001 --peers http://<IP-A>:8000,http://<IP-C>:8002
```

### C
```bash
python3 node.py --id C --port 8002 --peers http://<IP-A>:8000,http://<IP-B>:8001
```

## Use the client
Send PUT, GET, or check STATUS:
```bash
# Put value 1 on Node A
python3 client.py --node http://<IP-A>:8000 put x 1
# Retrieve value from Node B
python3 client.py --node http://<IP-B>:8001 get x
# Check the current state on Node C
python3 client.py --node http://<IP-C>:8002 status
```

## Lab Experiments / Scenarios

### 1. Delay / Reorder (Scenario A)

- Introduce an artificial delay inside the replicate_to_peers() function before sending to a specific peer:

```bash
time.sleep(delay_s)  # simulate network delay to 5 second for A -> C
```

- Observation: Node B may apply the update before Node C.
- Note: Lamport clocks still preserve logical ordering despite message delays.

---

### 2. Concurrent Writes (Scenario B)

- Send two PUT requests to the same key nearly simultaneously:

```bash
python3 client.py --node http://<IP-A>:8000 put x 1
python3 client.py --node http://<IP-B>:8001 put x 2
```

- Conflict resolution: Uses Lamport timestamps. The update with the higher timestamp wins.
- Result: All nodes eventually converge to the same value.

---

### 3. Temporary Outage (Scenario C)

- Stop a node (e.g., Node C) to simulate a temporary failure:

# On EC2, stop Node C (Ctrl+C in terminal running node.py)

- Perform updates on Node A and Node B while Node C is offline.
- Restart Node C:

```bash
python3 node.py --id C --port 8002 --peers http://<IP-A>:8000,http://<IP-B>:8001
```
- Node C participates in new updates and eventually converges with peers.
- Missed updates during downtime are not automatically recovered in the starter code.

---

### Conflict Resolution

- Last-Writer-Wins (LWW) using Lamport timestamp
- Tie-breaker: origin ID (lexicographic order)
- Optional: implement vector clocks or more advanced concurrency detection

