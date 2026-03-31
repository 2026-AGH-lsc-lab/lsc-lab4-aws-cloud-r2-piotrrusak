# AWS Cloud Lab — Serverless vs Containers: Latency and Cost Comparison

## 1. Objective

The objective of this lab was to compare three AWS execution environments:
- AWS Lambda (zip and container image)
- AWS Fargate (ECS)
- AWS EC2

The comparison focuses on:
- Latency (warm vs cold start)
- Throughput
- Burst performance
- Cost efficiency

---

## 2. Experimental Setup

### Application
A k-NN (k-nearest neighbors) search workload implemented in Python was deployed identically across all environments.

### Infrastructure Configuration

| Environment        | Configuration                  |
|------------------|------------------------------|
| Lambda           | 512 MB memory                 |
| Fargate          | 0.5 vCPU, 1 GB RAM           |
| EC2              | t3.small (2 vCPU, 2 GB RAM)  |

### Tools Used
- `curl` — functional testing
- `oha` — load and stress testing
- AWS CLI — Lambda invocation

---

## 3. Results

### 3.1 Warm Latency

| Environment          | Latency (ms) |
|----------------------|-------------|
| EC2                  | 26.25       |
| Fargate              | 23.49       |
| Lambda (zip)         | 52.51       |
| Lambda (container)   | 67.95       |

---

### 3.2 Cold Start Latency

| Environment          | Latency (ms) |
|----------------------|-------------|
| Lambda (zip)         | 79.40       |
| Lambda (container)   | 71.03       |

Cold start overhead is relatively small (~20–30 ms increase).

---

### 3.3 Throughput (200 requests, concurrency = 20)

| Environment | Requests/sec | Avg Latency |
|------------|--------------|------------|
| EC2        | 40.93        | 0.47 s     |
| Fargate    | 12.48        | 1.55 s     |

---

### 3.4 Burst Performance (500 requests, concurrency = 100)

| Environment | Requests/sec | Avg Latency | p95 Latency |
|------------|--------------|------------|-------------|
| EC2        | 49.90        | 1.85 s     | ~2.19 s     |
| Fargate    | 12.84        | 7.08 s     | ~8.00 s     |

---

## 4. Cost Model

### Lambda
- Pay-per-use pricing
- ~60 ms execution time
- Approximate cost per request:
  **~$0.00005**

### EC2
- Fixed cost:
  **~$0.0208 per hour**

### Break-even Analysis

The break-even point between Lambda and EC2 is approximately:

> **100–150 requests per second**

- Below this → Lambda is more cost-efficient
- Above this → EC2 becomes more economical

---

## 5. Analysis

### Latency
- EC2 and Fargate provide the lowest latency (~23–26 ms)
- Lambda introduces moderate overhead (~50–70 ms)
- Cold starts are noticeable but not severe

### Throughput
- EC2 achieves the highest throughput
- Fargate performs significantly worse under load
- Lambda scales automatically but is harder to benchmark with traditional tools

### Burst Behavior
- EC2 handles burst traffic efficiently
- Fargate shows severe latency degradation (~7 seconds average)
- Lambda benefits from automatic scaling

### Cost Efficiency
- Lambda is optimal for low and unpredictable workloads
- EC2 is optimal for high, sustained workloads
- Fargate offers simplicity but poor performance-to-cost ratio in this scenario

---

## 6. Conclusion

- **AWS Lambda** is the best choice for low or irregular workloads due to scalability and low cost.
- **AWS EC2** provides the best performance and becomes most cost-efficient at scale.
- **AWS Fargate** simplifies infrastructure management but showed the weakest performance in this experiment.

---

## 7. Final Recommendation

- Use **Lambda** for event-driven or low-traffic applications
- Use **EC2** for high-performance, high-throughput systems
- Use **Fargate** only when ease of management outweighs performance concerns

---

## 8. Notes

- Cold start measurements were obtained after a long idle period (~40 minutes)
- All environments used the same application logic for fair comparison
- Results may vary depending on AWS region and load conditions