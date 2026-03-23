# Containerized Data Pipeline: Real-Time API Streaming with Kafka, Spark, Airflow & PostgreSQL

Dockerized pipeline that ingests API data via Kafka, transforms it with Spark, stores it in PostgreSQL, and schedules everything through Airflow.

---

## What This Project Does

This pipeline pulls random user data from an external API, pushes it through a message queue, processes it, and lands it in a relational database. The entire workflow is automated and containerized.

Here is how data flows through the system:

```
External API → Kafka Producer → Kafka Topic → Spark Consumer → PostgreSQL
                                      ↑
                              Airflow (schedules everything)
```

**In plain English:**
1. A Python script hits the [Random User API](https://randomuser.me/) and streams the response into a Kafka topic
2. A Spark job picks up messages from that Kafka topic, cleans/structures the data, and writes it to PostgreSQL
3. Airflow ties both steps together. It triggers the streaming first, waits for it to finish, then kicks off the Spark job
4. Everything runs inside Docker containers, so there is nothing to install manually

---

## Tech Stack

| Tool | Role |
|------|------|
| **Apache Kafka** | Message broker that buffers data between ingestion and processing |
| **Apache Spark** | Distributed processing engine that reads from Kafka, transforms, and writes to Postgres |
| **Apache Airflow** | Workflow orchestration tool that schedules and monitors the pipeline |
| **PostgreSQL** | Storage layer and final destination for processed data |
| **Docker & Docker Compose** | Containerization layer that runs the entire stack with one command |
| **Python** | Glue code for the Kafka producer, Spark jobs, and Airflow DAGs |

---

## Project Structure

```
├── airflow/                 # DAGs and Airflow configuration
├── data/                    # Sample/reference data
├── scripts/                 # Utility and setup scripts
├── spark/                   # Spark job for consuming Kafka and writing to Postgres
├── src/                     # Kafka producer and constants
├── docker-compose.yml       # Main services (Kafka, Spark, Postgres)
├── docker-compose-airflow.yaml  # Airflow services
└── requirements.txt         # Python dependencies
```

---

## How to Run

### Prerequisites
- Docker and Docker Compose installed on your machine

### Steps

**1. Start the core services (Kafka, Spark, PostgreSQL):**
```bash
docker-compose up -d
```

**2. Start Airflow:**
```bash
docker-compose -f docker-compose-airflow.yaml up -d
```

**3. Access Airflow UI:**
Open `http://localhost:8080` in your browser and trigger the DAG.

---

## Architecture Decisions

**Why Kafka instead of hitting the API directly from Spark?**
Kafka decouples ingestion from processing. If Spark goes down, messages queue up in Kafka instead of being lost. In production, this pattern handles backpressure and allows multiple consumers.

**Why Airflow instead of a cron job?**
Airflow gives you dependency management between tasks, retry logic, and a UI to monitor pipeline health. A cron job can trigger things, but it cannot say "only run Spark after Kafka finishes successfully."

**Why Docker Compose?**
Five services with specific version requirements and network configurations. Docker Compose makes this reproducible across any machine with a single command.

---

## What I Would Improve

- [ ] Add data validation/quality checks between Kafka and Spark
- [ ] Set up a monitoring dashboard (Grafana) for pipeline metrics
- [ ] Add error handling and dead letter queue for failed messages
- [ ] Implement schema registry for Kafka to enforce data contracts
- [ ] Add unit tests for the Spark transformations

---

## Acknowledgments

Inspired by [HamzaG737's data engineering project](https://github.com/HamzaG737/data-engineering-project). Built upon and modified for learning purposes.
