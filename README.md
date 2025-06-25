
# 🚀 PostgreSQL to Microsoft Fabric Data Warehouse Integration (via Airbyte)

This project documents the full setup and configuration to extract data from a local PostgreSQL instance and load it into **Microsoft Fabric Data Warehouse** using [Airbyte](https://airbyte.com/).

---

## 📌 Table of Contents

1. [Project Overview](#project-overview)  
2. [Technologies Used](#technologies-used)  
3. [Project Setup](#project-setup)
   * Dockerized PostgreSQL  
   * Populating Data  
4. [Airbyte Setup](#airbyte-setup)  
   * Local vs Cloud  
5. [Connecting PostgreSQL as a Source](#connecting-postgresql-as-a-source)  
   * Local connection  
   * Cloud connection with ngrok  
6. [Connecting Microsoft Fabric as a Destination](#connecting-microsoft-fabric-as-a-destination)  
7. [Expected End State](#expected-end-state)  
8. [Troubleshooting](#troubleshooting)  

---

## 📝 Project Overview

This project demonstrates a full EL (Extract-Load) pipeline:

* Data is generated and stored in a local **PostgreSQL** database.  
* Airbyte extracts data from Postgres and loads it into **Microsoft Fabric Data Warehouse**.  
* Ideal for building modern data stacks or prototyping data ingestion pipelines.  

---

## 🛠️ Technologies Used

| Tool             | Purpose                            |
| ---------------- | ---------------------------------- |
| Docker           | Run PostgreSQL container           |
| PostgreSQL       | Source database                    |
| Airbyte          | EL pipeline platform               |
| Microsoft Fabric | Destination data warehouse         |
| Ngrok            | Tunnel Postgres to Airbyte Cloud   |

---

## 🔧 Project Setup

### 1. Create Local PostgreSQL with Docker

```bash
mkdir Airbyte_demo && cd Airbyte_demo

# Create docker-compose.yml
touch postgres_docker.yml
nano postgres_docker.yml
```

Paste in:

```yaml
services:
  postgres_demo:
    image: postgres:16
    container_name: postgres_demo
    ports:
      - "5433:5432"
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydatabase
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata: {}
```

Start the container:

```bash
docker compose -f postgres_docker.yml up -d
```

---

### 2. Populate the PostgreSQL Database

Access the container:

```bash
docker exec -it postgres_demo psql -U myuser -d mydatabase
```

Create table and add test data:

```sql
CREATE TABLE blog_posts (id serial not null, data jsonb);

INSERT INTO blog_posts VALUES (1,'{"title": "sunt aut facere repellat provident occaecti excepturi optio reprehend erit", "body": "quia at suscipit\nsuscipit"}'),
(2,'{"title": "madara aut facere repellat provident occaecti excepturi optio reprehend erit", "body": "wake up to reality uia at suscipit\nsuscipit"}'),
(3,'{"title": "bonolo molele aut facere repellat provident occaecti excepturi optio reprehend erit", "body": " we dont all have to exist uia at suscipit\nsuscipit"}'),
(4,'{"title": "givz beku aut facere repellat provident occaecti excepturi optio reprehend erit", "body": "get money uia at suscipit\nsuscipit"}'),
(5,'{"title": "uzuku aut facere repellat provident occaecti excepturi optio reprehend erit", "body": "the greatest hero quia at suscipit\nsuscipit"}'),
(6,'{"title": "eren aut facere repellat provident occaecti excepturi optio reprehend erit", "body": "keep moving forward quia at suscipit\nsuscipit"}'),
(7,'{"title": "monkey d luffy aut facere repellat provident occaecti excepturi optio reprehend erit", "body": "the one piece is real at suscipit\nsuscipit"}');
```

Exit:

```sql
\q
```

---

## 🧭 Airbyte Setup

### Option A: Use Airbyte Cloud

Go to: [https://airbyte.com/](https://airbyte.com/)

> Cloud version **cannot access localhost**, so you'll need to tunnel your local Postgres using `ngrok`.

Install and configure ngrok:

```bash
sudo snap install ngrok
ngrok config add-authtoken YOUR_NGROK_TOKEN
ngrok tcp 5433
```

Use the output like:

```
tcp://0.tcp.ngrok.io:12345
```

Use:

* Host: `0.tcp.ngrok.io`  
* Port: `12345`  
* Database: `mydatabase`  
* Username: `myuser`  
* Password: `mypassword`  

---

### Option B: Run Airbyte Locally (No ngrok needed)

Clone and start:

```bash
git clone https://github.com/airbytehq/airbyte.git
cd airbyte
./run-ab-platform.sh
```

Access via: [http://localhost:8000](http://localhost:8000)

Add the Postgres source with:

* Host: `localhost`  
* Port: `5433`  
* DB, User, Pass as above  

---

## 🧩 Connecting Microsoft Fabric as a Destination

Microsoft Fabric can be used as a destination via its **Synapse Data Warehouse endpoint** using the **T-SQL interface** (formerly Synapse SQL). Airbyte supports destinations using **Microsoft SQL Server**, which is compatible.

### Steps:

1. In Airbyte, add a new destination.
2. Choose **Microsoft SQL Server** (which works with Microsoft Fabric's SQL endpoint).
3. Enter the following:

   * **Host**: The Fully Qualified Domain Name (FQDN) of your Fabric Warehouse endpoint  
   * **Port**: Usually `1433`  
   * **Database**: The name of your Fabric Warehouse  
   * **Schema**: E.g., `dbo`  
   * **Username**: Your Fabric SQL user  
   * **Password**: Corresponding password  
   * **SSL**: Enable if your endpoint requires encrypted communication

⚠️ Make sure:

* Your workspace in Microsoft Fabric is configured to allow external connections.
* You are using the **SQL endpoint URL**, not Power BI or OneLake endpoints.
* The account used has appropriate permissions for writing to the warehouse.

---

## 🎯 Expected End State

* You have a running local PostgreSQL database with sample data.  
* Airbyte connects to this Postgres instance as a source.  
* Microsoft Fabric Data Warehouse is configured as the destination.  
* Data from Postgres is synced into Microsoft Fabric on-demand or on a schedule.  

---

## 🧯 Troubleshooting

| Issue                                      | Fix                                                                 |
| ------------------------------------------| ------------------------------------------------------------------- |
| `connection refused` on localhost         | Airbyte Cloud can't reach local DB — use ngrok tunnel               |
| `ngrok` errors about credit card          | Add a card to ngrok to enable TCP tunnels (won't be charged)        |
| Port conflict (e.g. 5432 in use)          | Use a different host port, e.g. 5433                                |
| Microsoft Fabric connection fails         | Verify SQL endpoint, user permissions, and firewall settings        |
| Destination option not visible            | Use **Microsoft SQL Server** for Fabric SQL endpoint compatibility |

---

## 🧾 Credits

This project was built step-by-step using:

* [Airbyte Documentation](https://docs.airbyte.com/)  
* [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)  
* [ngrok](https://ngrok.com/)  
* [Microsoft Fabric Docs](https://learn.microsoft.com/fabric)  
