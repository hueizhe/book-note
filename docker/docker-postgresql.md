
~~~docker
docker run --name postgresq01 -e POSTGRES_PASSWORD=password -p 5432:5432   -d postgres:9.4
~~~ 
-v /etc/data/pgdata:/var/lib/postgresql/data-d