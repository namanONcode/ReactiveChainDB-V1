# **ReactiveChainDB-V1**

Modern distributed systems require increasingly secure, low-latency, and high-throughput transaction handling—particularly in healthcare, finance, and real-time scheduling.  
Conventional blockchain architectures often lack the performance efficiency needed for such reactive environments.

To address this, a **high-throughput reactive microservice framework** extending **BigchainDB** is proposed for decentralized and scalable transaction processing.

---

# **Problem 1: CPU and Queue Congestion Under Load**

### **The Hurdle**  
During high-traffic simulations (1,000 requests in batches), the system hit a bottleneck in **Batch 5**, where a temporary slowdown caused the **maximum response time to spike to 4.37 seconds** due to CPU and queue congestion.

### **Solution**  
We implemented a **Parallel Backpressure-Handling Algorithm** using `Sinks.Many`.

- The reactive buffer was dynamically reconfigured and resized at runtime to adapt to fluctuating workloads.  
- Leveraging **Non-Blocking I/O** combined with reactive backpressure allowed the system to recover quickly.  
- After the temporary surge, response times stabilized back to the **30–70 ms** range.

![image](https://assets.devfolio.co/content/9285a1bb63944d76aee21933e63c35bb/17dea845-dc84-4653-bffa-0d47ae0dd6d0.png)

---

# **Problem 2: Complexity of Blockchain Integration**

### **The Hurdle**  
Hyperledger Fabric was initially explored due to its modular design.  
However, its complexity conflicted with the goal of creating a **lightweight, reactive integration**.

### **Solution**  
We pivoted to **BigchainDB**, which offered a lighter and more reactive-friendly framework.

To further mitigate slowness from direct blockchain queries, we developed a **hybrid caching layer using Redis**:

- Instead of querying BigchainDB on each read,  
- We cached transaction URLs in Redis mapped to User IDs  
- Allowing *instant* lookups and minimal blockchain load.

![image](https://assets.devfolio.co/content/9285a1bb63944d76aee21933e63c35bb/a8a4485d-da13-4eb0-bb34-2a4e10fb9c21.png)

---

# **Problem 3: Handling Real-Time Data Updates**

### **The Hurdle**  
Traditional REST API polling was too slow and inefficient for low-latency environments such as **healthcare**, **finance**, and **real-time systems**.  
This caused delays in propagating fresh data to clients.

### **Solution**  
We built a **Redis Watcher Service + WebSockets** architecture.

- The service watches for specific User ID updates in the Redis cache  
- On change, it triggers an immediate request to BigchainDB  
- The updated data is **pushed instantly** to the client via WebSocket  
- Completely eliminating the need for repeated polling

![image](https://assets.devfolio.co/content/9285a1bb63944d76aee21933e63c35bb/b7d8ad3e-245d-413c-a615-cdc49434c61c.png)
