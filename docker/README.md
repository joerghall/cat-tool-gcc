BUild docker

docker build -t jhhj/cbt-gcc-centos7:1 -f docker/centos.docker  docker

docker push jhhj/cbt-gcc-centos7:1

docker run -it jhhj/cbt-gcc-centos7:1 /bin/bash
