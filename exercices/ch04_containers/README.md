- build the image
```bash
    docker build -t nodejs-hello-world .
```
- Publish the api on host port 80
```
    docker run -p 80:3000 nodejs-hello-world
```
keep in mind that the first port is the host port and the second is the container port
