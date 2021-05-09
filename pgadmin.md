##### load pgadmin image
```shell
docker pull dpage/pgadmin4
```

##### run pgadmin
```shell
docker run -p 5050:80 \
    -e "PGADMIN_DEFAULT_EMAIL=user@domain.com" \
    -e "PGADMIN_DEFAULT_PASSWORD=SuperSecret" \
    -d dpage/pgadmin4
```
