# SqlServerOnDocker
Proof of concept project with Microsoft SQL Server and Django Framework setup on docker containers.

## config notes:
1. By default, .env file is used for configuration
2. To use config for other environments, add the following option to your commands: `--env-file ./docker/config/.env.live`. For example: `docker compose --env-file ./docker/config/.env.live up mykeycloak`

## General (see https://docs.docker.com/compose/gettingstarted/)
1. start up the application by running `docker compose up`
2. If you want to run your services in the background, you can pass the -d flag (for “detached” mode) to `docker compose up -d` and use `docker compose ps` to see what is currently running.
3. If you started Compose with docker compose up -d, stop your services once you’ve finished with them: `docker compose stop`
4. The docker compose run command allows you to run one-off commands for your services. For example, to see what environment variables are available to the mykeycloak service: `docker compose run myredis env`
5. Use `docker compose down --volumes` to bring everything down, removing the containers entirely, with the down command. Pass --volumes to also remove the data volume used by the containers.
6. Compose uses a project name to isolate environments from each other. The default project name is the basename of the project directory. You can set a custom project name by using the -p command line option or the COMPOSE_PROJECT_NAME environment variable.

## To start development:
1. install [docker](https://docs.docker.com/#/components) and [docker-compose](https://docs.docker.com/compose/install/)
2. clone this repository
3. run `docker-compose build db` to build db container
4. run `docker-compose up -d db` to run SQL Server container in detached mode in the background
5. run `docker-compose run db sqlcmd -S db1.internal.prod.example.com -U SA -P '<YourStrong@Passw0rd>' -Q  "RESTORE FILELISTONLY FROM DISK = N'/var/opt/mssql/backup/AdventureWorksDW2017.bak'"`
    to verify database file names before restore,
6. run `docker-compose run db sqlcmd -S db1.internal.prod.example.com -U SA -P '<YourStrong@Passw0rd>' -Q  "RESTORE DATABASE AdventureWorksDW2017 FROM DISK = N'/var/opt/mssql/backup/AdventureWorksDW2017.bak' WITH MOVE 'AdventureWorksDW2017' TO '/var/opt/mssql/data/AdventureWorksDW2017.mdf', MOVE 'AdventureWorksDW2017_log' TO '/var/opt/mssql/data/AdventureWorksDW2017_log.ldf' "`
    to restore AdventureWorksDW2017 database on SQL Server
7. run `docker-compose run web python3 manage.py migrate` to apply migrations on default database. In this case it will be AdventureWorksDW2017.
8. run `docker-compose run web python3 manage.py createsuperuser` to create admin account

## keycloak
1. Run: `sudo docker-compose build mykeycloak`
2. Run: `docker-compose up -d mykeycloak`
3. Browse to: `localhost:8443` and login using `admin` and `change_me`

## To run project:
 
1. run `docker-compose up web`
2. point your browser to `localhost:8080`
3. press `CTRL+C` to stop

## Access to sql server
1. sudo docker-compose run db sqlcmd -S db1.internal.prod.example.com -U SA -P '<YourStrong@Passw0rd>' -Q 'select 1 from AdventureWorksDW2017'
2. sudo docker exec -it sqlserverondocker_db_1 bash


## Scaling a service
1. To run multiple instances of myredis, use `docker-compose up --scale myredis=3 myredis` at startup, or
2. At runtime using: `docker-compose scale myredis=3`
2. To configure port range for a scaled service, use `ports: - "6379-6385:6379"`