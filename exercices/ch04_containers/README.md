- build the image
```bash
    docker build -t nodejs-hello-world .
```
- Publish the api on host port 80
```bash
    docker run -p 80:3000 nodejs-hello-world
```
keep in mind that the first port is the host port and the second is the container port

- change the version of node. 
with the newer version, npm install fails looking for a package.json. Had to remove the npm install and it 
works without it.

- save an image to a tar and create an image from a tar. 
    * save image in tar
```bash
    docker save -o [name].tar [existing image name]
```
    * load image from tar 
``` bash
    docker load --input [name].tar
```

