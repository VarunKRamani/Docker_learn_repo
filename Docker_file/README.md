## Understanding Docker file.

**What** is a docker file ? --> A Dockerfile is a text document that contains steps to construct a container image for an application, which will be read by docker.

**Need for Docker** --> Simply to put, docker solved the problem. 'It works in my local machine/runs in my local machine but is broken on the Server' problem. Docker solves this issue by providing:
- Build once, Run anywhere: Docker packages your application along with everything it needs (code, runtime, system tools, and libraries) into a single container. That container will run identically on our local machine or on a cloud server.

- Fast and consistent: Containers start up in seconds because they share host's machines kernal, making them much lighter and faster then traditional virtual machines.
 
- Environment Insolation: Docker containers keeps applications separate from each other and from the host machine, one can run two different applications with conflicting versions of tech stack on the same server without any interference. 
______
**Process of writing a docker file**

A dockerfile is a sequential lines of instructions or a linear script.
- The Base Image : Dockerfile starts with a `FROM` instruction.
 What `FROM` does --> The `FROM` instruction tells Docker which existing template to download and start with. Docker has a public hub (Docker Hub) with ready-to-use base images maintained by official groups. 
