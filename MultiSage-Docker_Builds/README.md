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

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp .

#stage 2 - Run stage ------------------------------------

FROM alpine:latest
WORKDIR /root/

COPY --from=builder /app/myapp .

EXPOSE = 8080
CMD ["./myapp"]

```
