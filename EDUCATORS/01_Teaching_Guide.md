# KANTOR_K5 — Educator's Teaching Guide

## Course Fit: distributed systems, consensus protocols, orchestration frameworks

## 3-Week Module: Distributed AI Orchestration with KANTOR_K5

### Week 1: Orchestration Fundamentals
**Lecture Topics:**
- What KANTOR_K5 orchestrates: agents, models, and data pipelines
- The K5 topology: 5-node consensus for fault-tolerant task assignment
- Why Raft consensus matters for AI workload distribution
- Comparing KANTOR to Kubernetes for AI-specific workloads

**Lab Exercise:**
```python
from kantor import K5Cluster, Node, Task
cluster = K5Cluster(nodes=[
    Node("node-1", capacity=2),
    Node("node-2", capacity=2),
])
task = Task(name="embed_documents", payload={"docs": ["doc1.txt", "doc2.txt"]})
result = cluster.submit(task)
print(f"Assigned to: {result.assigned_node}, status: {result.status}")
```

### Week 2: Task Scheduling and Fault Tolerance
**Lecture Topics:**
- Priority queues and preemption in KANTOR
- Heartbeat monitoring and node failure recovery
- Idempotent task execution: why it matters for retry logic
- Resource-aware scheduling: GPU vs. CPU task routing

**Lab Exercise:**
```python
from kantor import K5Cluster, Task, Priority
cluster = K5Cluster.from_config("kantor_config.yaml")
# Submit high-priority inference task
inference_task = Task("llm_inference", priority=Priority.HIGH,
                      resource_hint="gpu",
                      payload={"prompt": "Summarize this document"})
result = cluster.submit(inference_task, timeout=30)
print(result.output)
```

### Week 3: Integration — KANTOR_K5 in the Anticloud Stack
**Lecture Topics:**
- KANTOR as the scheduler underneath PAX_SCHEDULER
- Connecting KANTOR to AIOSS for ledger tracking
- Scaling KANTOR on Kaggle multi-GPU notebooks
- Monitoring with PAX_MONITOR

**Lab Exercise:**
```python
from kantor import K5Cluster
from aioss_format import AIOSSLedger
ledger = AIOSSLedger()
cluster = K5Cluster.from_config("kantor_config.yaml", ledger=ledger)
cluster.run_pipeline("inference_pipeline.yaml")
ledger.export("kantor_session.aioss.json")
```

## Exam Questions
1. Explain Raft consensus in the context of KANTOR_K5. How does the leader election process prevent split-brain in a 5-node cluster?
2. Why must task execution be idempotent in a distributed orchestration system? Give a concrete example of what happens without idempotency on node failure.
3. Compare KANTOR_K5's GPU-aware scheduling to Kubernetes device plugins. What does KANTOR optimize for that generic Kubernetes does not?
