# KANTOR_K5 — Student Getting Started

## What You'll Build
A distributed task orchestration cluster that schedules AI workloads across nodes, with fault tolerance and GPU-aware routing.

## Prerequisites
- Python 3.10+
- Understanding of async programming and basic distributed systems concepts

## Install
```bash
pip install kantor-k5
```

## First Working Example
```python
from kantor import K5Cluster, Node, Task

# Create a single-node cluster for learning
cluster = K5Cluster(nodes=[Node("local", capacity=4)])

# Submit a simple task
task = Task(
    name="text_summary",
    payload={"text": "The Anticloud is a sovereign AI infrastructure platform."}
)
result = cluster.submit(task)
print(f"Status: {result.status}")
print(f"Output: {result.output}")
```

## Multi-Node Cluster
```python
from kantor import K5Cluster, Node, Task, Priority

cluster = K5Cluster(nodes=[
    Node("node-1", capacity=2, resource="cpu"),
    Node("node-2", capacity=2, resource="gpu"),
])

# High priority GPU task
task = Task("llm_inference", priority=Priority.HIGH, resource_hint="gpu",
            payload={"prompt": "Summarize this paragraph"})
result = cluster.submit(task, timeout=30)
print(result.output)
```

## On Kaggle (loiskleinner account, T4 GPU)
```python
!pip install kantor-k5
# Kaggle T4 is single-node but you can simulate multi-node
from kantor import K5Cluster, Node
cluster = K5Cluster(nodes=[Node("kaggle-t4", capacity=4, resource="gpu")])
```

## What's Next
- Try submitting 10 tasks and observing load balancing
- Simulate a node failure by removing a node mid-run
- See the EDUCATORS guide for Raft consensus details
