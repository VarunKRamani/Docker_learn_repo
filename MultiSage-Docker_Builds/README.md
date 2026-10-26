## Understanding the need for Multistage Docker builds.

Multi-stage builds were introduced to solve the problems by cleanly **separating the build environment from the runtime environment** within a single Dockerfile.

**A Multi-stage Dockerfile is a Dockerfile that uses multiple `FROM` statments to break the image build process inti separate, distinct stages.**

Simply put, it lets us use a heavy image with all our compilers and tools to build our app, and then copy only the finished, compiled app into a tiny, lightweight image to run it in the production. 


