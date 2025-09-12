# Velox/Gluten Performance Demo on S3

A Docker Compose template demonstrating Apache Spark performance improvements with Velox and Gluten acceleration on S3 storage.

## Overview

This repository provides a complete Spark cluster setup with:
- **Apache Spark 3.5.2** with Hadoop 3.4.1
- **Gluten 1.4.0** with Velox backend for query acceleration
- **JupyterLab** environment for interactive analysis
- **S3A integration** with optimized committers and native Hadopp libraries
- **Performance benchmarking** notebook comparing Velox vs vanilla Spark

## Architecture

- **Spark Master**: Cluster coordinator (port 8080)
- **Spark Workers**: 2 worker nodes (4 cores, 4GB RAM each)
- **Spark History Server**: Job monitoring (port 18080)
- **JupyterLab**: Interactive notebook environment (port 8888)

## Requirements

- **Hardware**: x86_64 machines (tested on EC2 c5.2xlarge)
- **Software**: Docker & Docker Compose
- **AWS**: S3 bucket with read/write permissions

## Quick Start

1. **Clone and build**:
```bash
git clone <repository-url>
cd spark-gluten-velox
docker-compose up --build
```

2. **Access JupyterLab**:
   - Open http://localhost:8888
   - No token required (development setup)
   - Upload `test.ipynb` via the JupyterLab interface

3. **Configure S3 access**:
   - **EC2 (Recommended)**: Attach an IAM role to your EC2 instance with S3 read/write permissions
   - **Local/Other**: Set AWS credentials via environment variables or AWS CLI
   - Update S3 bucket name in the test notebook
   - Ensure your S3 bucket allows read/write access from your AWS account/role

4. **Run performance tests**:
   - Open `test.ipynb` in JupyterLab
   - Execute cells to compare Velox vs vanilla Spark performance

## Performance Test

The included benchmark generates 50M synthetic records and runs three query types:

- **Query A**: Multi-aggregation with groupBy (hash aggregations, projections, filters)
- **Query B**: Rollup operations (cubing-style analytics)  
- **Query C**: Star schema joins with top-K sorting

Each test runs twice: once with Velox enabled, once with vanilla Spark, measuring execution time differences.

## Expected Performance Gains

Results vary based on data size, query complexity, and hardware but you should see here some performance gains.

## Configuration

### Spark Configuration
Key Velox settings in the notebook:
```python
.config("spark.plugins", "org.apache.gluten.GlutenPlugin")
.config("spark.memory.offHeap.enabled", "true")
.config("spark.memory.offHeap.size", "8g")
.config("spark.shuffle.manager", "org.apache.spark.shuffle.sort.ColumnarShuffleManager")
```

### S3 Integration
- Uses S3A connector with AWS SDK v2
- Magic committers enabled for better write performance
- Hadoop 3.4.1 for latest S3 optimizations with native libraries

### AWS Permissions

You need to have permissions for your chosen S3 bucket.

**For EC2 instances (recommended approach):**
1. Create an IAM role with S3 permissions:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::your-bucket-name",
                "arn:aws:s3:::your-bucket-name/*"
            ]
        }
    ]
}
```
2. Attach the role to your EC2 instance
3. No additional credential configuration needed

**For local development:**
- Configure AWS CLI: `aws configure`
- Or set environment variables: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`

## Monitoring

- **Spark Master UI**: http://localhost:8080
- **Worker UIs**: http://localhost:8081, http://localhost:8082
- **History Server**: http://localhost:18080
- **Spark Driver UI**: http://localhost:4040 (during job execution)

## Troubleshooting

**Build Issues**:
- Ensure Docker has sufficient memory (8GB+ recommended)
- Check internet connectivity for dependency downloads

**Runtime Issues**:
- Verify AWS credentials are properly configured
- Ensure S3 bucket exists and has correct permissions
- Check Docker logs: `docker-compose logs <service-name>`

**Performance Issues**:
- Adjust worker memory/cores in docker-compose.yaml
- Modify data size (N_ROWS) in notebook for your hardware
- Monitor resource usage during execution (you can go to 4040 to see Spark UI)

## Customization

- **Scale workers**: Modify `SPARK_WORKER_CORES` and `SPARK_WORKER_MEMORY` in docker-compose.yaml
- **Add workers**: Duplicate worker service with different ports
- **Change data size**: Adjust `N_ROWS` variable in the benchmark notebook
- **Custom queries**: Add your own workloads to the notebook
- **Improve the docker-compose**: The docker compose can be improved in order spark-history server to work well ( you will need to send logs to a shared volume or to s3). 
