### install and launch 8000e SONiC from scratch

1. Download tarballs to a common directory: http://vxr8000.cisco.com/8000/eft7.8/sonic/
```
cisco@ubuntu:~/images/8000-eft7.8$ ls ~/images/
8000-emulator-eft7.8.tar  8000-sonic-eft7.8.tar
```

2. cd into the directory and untar using this command:
```
find . -name '*.tar' -exec tar -xvf {} \;
```

3. cd into the newly created 8000-eft.x.y directory and run the docker build image shell script:
```
cd 8000-eft7.8/
./scripts/build_docker_image.sh 8000:7.8 ./docker/sonic/Dockerfile 
```

4. the build script takes a few minutes. successful build will end with something like:
```
=> exporting to image                                                                                           28.0s
=> => exporting layers                                                                                          28.0s
=> => writing image sha256:8c0650ec1724fe8d55ce4d9b4fc0cb075dd8b0b0361ac2196ecc42e62b5aa272                      0.0s
=> => naming to docker.io/library/8000:7.8                                                                       0.0s
```

5. Verify docker image with 'docker image ls':
```
cisco@ubuntu:~/images/8000-eft7.8$ docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
8000         7.8       8c0650ec1724   10 minutes ago   11.3GB
```

6. 
   