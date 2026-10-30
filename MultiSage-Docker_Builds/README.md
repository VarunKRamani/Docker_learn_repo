# Understanding the need for Multistage Docker builds.

Multi-stage builds were introduced to solve the problems by cleanly **separating the build environment from the runtime environment** within a single Dockerfile.

**A Multi-stage Dockerfile is a Dockerfile that uses multiple `FROM` statments to break the image build process inti separate, distinct stages.**

Simply put, it lets us use a heavy image with all our compilers and tools to build our app, and then copy only the finished, compiled app into a tiny, lightweight image to run it in the production. 

- **Separation of Concerns**: We use a heavy, fully-loaded container just for compiling and testing your code, and a separate, ultra-lightweight container (like Alpine, Scratch, or Distroless) just for running it.

- **Enhanced Security**: Stripping out build tools drastically reduces our container's attack surface and vulnerability count.

- **Lightning-Fast Deployments**: Smaller images (20MB instead of 1GB+) transfer across networks instantly, speeding up deployments and auto-scaling.

- **Minimalist Production Artifacts**: We copy only the compiled binary or final asset across stages, throwing away gigabytes of compilers and source code.

___

## Writing a Multi-stage Dockerfile

A Multi-Stage Dockerfile mainly has two stages, namely: **Build stage** and **Runtime stage**

**Build Stage**: Starting with fully loaded base image with compliers, tools and label it using `AS Builder`. Copy the Source code and run compilation or  packaging commands in this stage.

**Example**: `Golang:1.23-alpine AS Builder`

**Runtime Stage**: Create a new block with tiny and clean base image. Use `COPY --from=Builder` to only copy the finished binary or file into this clean image and define the execution commands like `CMD`. 

**Example**: `FROM alpine:latest`

___

**A Multi-stage Dockerfile for a Go application**

```
#Stage 1 - build stage --------------------------------
FROM golang:1.23-alpine AS builder

# Set the work directory
WORKDIR /app

# Download the dependencies
COPY go.mod go.sum ./
RUN go mod download

# Copy the source code
COPY . .

# Build the binary     
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

#stage 2 - Run stage ------------------------------------

# start fresh with lightweight alpine image
FROM alpine:latest

# Set the work directory 
WORKDIR /root/

# copy the compiled binary from the builder stage to this fresh image
COPY --from=builder /app/myapp .

# application listens for incoming traffic on port 8080
EXPOSE 8080

# Application starts running, and begins listening on the set port.
CMD ["./myapp"]
```
<img width="700" height="350" alt="GO_app_docker_file" src="https://github.com/user-attachments/assets/e83aa606-2dc9-4743-9ebf-bff87f449205" />

**Note**
- `CGO_ENABLED=0` --> Disables CGO, which is the mechanism Go uses to call C/C++ libraries. By setting this to 0, you force Go to create a 100% statically linked binary. This means the compiled executable contains everything it needs inside itself and doesn't rely on external C libraries (like libc) from the host operating system.

- `GOOS=linux` --> Tells the Go compiler to target the Linux operating system. If you ever build your Docker image from a Mac or Windows machine, this ensures the resulting binary is built specifically for Linux containers.

___
**Multi-Stage Dockerfile for a nodejs Application**

```
# --- STAGE 1: Build Stage ---
# Use a full Node image to install packages and build
FROM node:20-alpine AS builder
WORKDIR /app

# Copy package files and install all dependencies
COPY package*.json ./
RUN npm ci

# Copy source code and build (if using a bundler/TypeScript)
COPY . .
RUN npm run build

# --- STAGE 2: Runtime Stage ---
# Start fresh with a clean Node runtime image
FROM node:20-alpine AS runner
WORKDIR /app

# Copy package files and install ONLY production dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy the built output from the builder stage
COPY --from=builder /app/dist ./dist

EXPOSE 3000
CMD ["node", "dist/index.js"]
```
_____

**Multi-stage Dockerfile for a Java application**

```
# --- STAGE 1: Build Stage ---
# Use a heavy Maven + JDK image to compile the app
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /app

# Copy project configuration and source files
COPY pom.xml .
COPY src ./src

# Build the project and package into a JAR file (skipping tests)
RUN mvn clean package -DskipTests

# --- STAGE 2: Runtime Stage ---
# Start fresh with a lightweight JRE runtime image
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Copy ONLY the compiled JAR file from the builder stage
COPY --from=builder /app/target/myapp.jar app.jar

EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```
______
**A Multi-stage Dockerfile for a Python application**

```
# --- STAGE 1: Build the packages ---
FROM python:3.11-slim AS builder
WORKDIR /app

# Install dependencies into a separate folder (/opt/venv)
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements.txt .
RUN pip install -r requirements.txt

# --- STAGE 2: Run the app ---
FROM python:3.11-slim
WORKDIR /app

# Copy ONLY the installed packages from Stage 1
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy your actual Python code
COPY . .

EXPOSE 8000
CMD ["python", "main.py"]
```
