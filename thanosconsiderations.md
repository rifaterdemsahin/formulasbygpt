When implementing a Thanos receiver using a remote writer on Prometheus metrics, here are the top 8 questions to consider:

1. **Data Ingestion Rate**: 
   - What is the expected volume of metrics that will be ingested by the Thanos receiver from the remote writer? Can the Thanos receiver handle the incoming data rate?

2. **Sharding and Replication**: 
   - How will you implement sharding or replication to ensure availability and load balancing across multiple Thanos receivers? Will you use hash-based sharding or any other mechanism?

3. **Storage Backend**: 
   - Which object storage will you use for Thanos (e.g., S3, GCS, or Azure Blob)? Are you confident that the storage system is properly configured to handle high volumes of data efficiently?

4. **Latency Requirements**: 
   - What are the acceptable latencies for ingesting and querying metrics from Thanos? Will your network and configuration be able to meet these requirements?

5. **Resilience and Failover**: 
   - How will the system behave during a receiver failure? Will there be automatic failover mechanisms in place to ensure high availability?

6. **Remote Write Configuration**: 
   - How should the Prometheus remote writer be configured (e.g., batching, compression, timeout settings) to optimize performance without overloading the Thanos receiver?

7. **Security and Authentication**: 
   - What security measures are in place for communication between the Prometheus remote writer and Thanos receiver? Will you use TLS, authentication tokens, or other methods to secure data?

8. **Monitoring and Alerting**: 
   - How will you monitor the health of the Thanos receiver and the entire pipeline? What metrics and logs should be collected to ensure the system is running smoothly and to detect any potential issues early?

These questions should help guide your implementation strategy and ensure a robust Thanos receiver setup with Prometheus metrics.