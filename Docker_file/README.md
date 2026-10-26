## Understanding Docker file.

**What** is a docker file ? --> A Dockerfile is a text document that contains steps to construct a container image for an application, which will be read by docker.

**Need for Docker** --> Simply to put, docker solved the problem. 'It works in my local machine/runs in my local machine but is broken on the Server' problem. Docker solves this issue by providing:
- Build once, Run anywhere: Docker packages your application along with everything it needs (code, runtime, system tools, and libraries) into a single container. That container will run identically on our local machine or on a cloud server.

- Fast and consistent: Containers start up in seconds because they share host's machines kernal, making them much lighter and faster then traditional virtual machines.
 
- Environment Insolation: Docker containers keeps applications separate from each other and from the host machine, one can run two different applications with conflicting versions of tech stack on the same server without any interference. 
______
**Process of writing a docker file**

A dockerfile is a sequential lines of instructions or a linear script.
- **The Base Image**(**Foundation**) : Dockerfile starts with a `FROM` instruction.
 What `FROM` does --> The `FROM` instruction tells Docker which existing template to download and start with. Docker has a public hub (Docker Hub) with ready-to-use base images maintained by official groups.

**Example**: `FROM golang:1.23-alpine`

- **Environment & Working Directory Setup**(**workspace**) : This section prepared the containers internal environment before the files are copied or installed. Set the current working directory for actions like (COPY or RUN)

**Example**: `WORKDIR /APP`

- **Dependencies & File Ingestion (Build stage)** : 
This is where external dependencies are installed and source code is brought into the container. To maximize layer caching, dependency files are usually copied and installed before the main source code.

**Example**:
`COPY go.mod go.sum ./`
`RUN go mod download`

- **Runtime Configuration**: There instructions define how containers must behave once they are finally instantiated and run.
`EXPOSE`: Documents which network port the application listens on.
`CMD`: Defines the executable command that runs when the container starts.

**Example**:
`EXPOSE 8080`
`CMD ["./myapp"]`

______________
# A Dockerfile for a Go Application 

```
# 1. Starting with an official Go image to build the app
FROM golang:1.23-alpine

# 2. Create and set the working directory inside the container
WORKDIR /app

# 3. Copy dependency files first to take advantage of caching
COPY go.mod go.sum ./

# 4. Download all Go modules/dependencies
RUN go mod download

# 5. Copy the rest of your application source code into the container
COPY . .

# 6. Build the Go application into an executable binary file
RUN go build -o myapp .

# 7. Tell Docker which network port the application listens on
EXPOSE 8080

# 8. Define the default command to run when the container starts
CMD ["./myapp"]
```
____________
# A Dockerfile for a Python Application 

````
# 1. Start with an official Python base image
FROM python:3.11-slim

# 2. Set the working directory inside the container
WORKDIR /app

# 3. Copy the dependencies file into the container
COPY requirements.txt .

# 4. Install the Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# 5. Copy the rest of your application source code into the container
COPY . .

# 6. Tell Docker which network port the application listens on (change if needed, e.g., 5000 for Flask)
EXPOSE 8000

# 7. Define the default command to run your Python app
CMD ["python", "main.py"]
````
- `python:3.11-slim`: A lightweight version of python to keep the container size small and has everything needed to run python.
- `--no-cache-dir`: Tells `pip` to not keep downloaded packages within the container saving dick space.

_______
# **Issues with Single-Stage Builds**
When we use a single-stage Dockerfile, everything happens in one container environment from start to finish. This creates several major technical drawbacks:

- **Massive Image Bloat**: To build an app (like Go, Java, or Node.js), **we need heavy SDKs, compilers, package managers (npm, pip, mvn)**, and source code files. In a single-stage build, all of those **bulky tools remain baked into the final image**. A Go app that compiles down to a 20MB binary might end up sitting inside a 1GB+ container image because the entire Go SDK is dragged along.

- **Slow CI/CD Pipelines and Scaling**: Large image sizes take longer to build, longer to push/pull to container registries (like Docker Hub or ECR), and longer to download onto cloud nodes when Kubernetes or Docker Swarm tries to scale up pods. This introduces lag into your deployment pipeline.

- **Risk of Leaking Sensitive Files**: Single-stage builds often require copying our entire local directory (COPY . .) into the container. This can accidentally bake in sensitive files like local configuration configs, .env files, git history, or test scripts into your production artifact.

**Why We Need Multi-Stage Builds: Multi-stage builds were introduced to solve these exact problems by cleanly separating the build environment from the runtime environment within a single Dockerfile.**
