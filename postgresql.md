##### clone docker-library/postgres repository
```shell
git clone https://github.com/docker-library/postgres.git
```

##### build image
```shell
docker build 10/alpine -t postgres-10 --no-cache
```

##### create container
```shell
docker run -p 0.0.0.0:5432:5432/tcp \
	-e POSTGRES_PASSWORD=$POSTGRES_PASSWORD \
	-it postgres-10:latest
```
