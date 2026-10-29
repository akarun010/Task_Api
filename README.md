# Task API

A small Spring Boot 3 + MySQL REST API made for practicing deployment:
Docker, Docker Compose, AWS EC2 and RDS.

## Endpoints 

| Method | Path | Description |
|--------|------|-------------|
| GET | /api/tasks | List tasks |
| GET | /api/tasks/{id} | Get one task |
| POST | /api/tasks | Create a task |
| PUT | /api/tasks/{id} | Update a task |
| DELETE | /api/tasks/{id} | Delete a task |
| GET | /actuator/health | Health check |

Example body: `{ "title": "Learn RDS", "status": "TODO" }` (status: TODO, IN_PROGRESS, DONE)

## Configuration (environment variables)

| Variable | Meaning |
|----------|---------|
| DB_URL | JDBC URL, e.g. jdbc:mysql://host:3306/taskdb |
| DB_USER | Database user |
| DB_PASSWORD | Database password |

## Run locally with Docker Compose

```
cp .env.example .env      # then edit DB_PASSWORD
docker compose up --build
curl http://localhost:8080/actuator/health
```

## Deploy on AWS (EC2 + RDS)

1. Build the jar: `mvn package -DskipTests` (creates `target/task-api-0.0.1-SNAPSHOT.jar`).
2. Create an RDS MySQL database (free tier, **public access: No**) with an initial database named `taskdb`.
3. Create a security group for RDS allowing port **3306 only from the EC2 instance's security group**.
4. Launch an EC2 instance in the same VPC. Allow SSH (22) and the app port (8080) **only from your IP**.
5. Copy the jar: `scp -i key.pem target/task-api-0.0.1-SNAPSHOT.jar ubuntu@<ec2-ip>:~`
6. On EC2: `sudo apt update && sudo apt install -y openjdk-17-jre-headless`
7. Set variables and run (the password prompt keeps it out of shell history):

```
export DB_URL="jdbc:mysql://<rds-endpoint>:3306/taskdb"
export DB_USER="<master-username>"
read -s -p "DB password: " DB_PASSWORD; export DB_PASSWORD
java -jar task-api-0.0.1-SNAPSHOT.jar
```

8. Test from your PC: `curl http://<ec2-ip>:8080/actuator/health`, then use Postman on `/api/tasks`.

### Docker variant on EC2

Install Docker, copy or pull the image, then:

```
docker run -d -p 8080:8080 \
  -e DB_URL="jdbc:mysql://<rds-endpoint>:3306/taskdb" \
  -e DB_USER="<master-username>" \
  -e DB_PASSWORD="<password>" \
  task-api:1.0
```

## Clean up

Terminate the EC2 instance, delete the RDS database (and any snapshots), and remove unused security groups.
