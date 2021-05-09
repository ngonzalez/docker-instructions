##### build image
```
docker build 10/alpine -t postgres --no-cache
```

##### create container
```
docker run -p 0.0.0.0:5432:5432/tcp \
	-e POSTGRES_PASSWORD='X3/eLR6CDtgbaPUXMd7WIvZ3Aic6K9aJcrbbQW9bUBtzEJVxODvUhPNK' \
	-it postgres:latest
```

