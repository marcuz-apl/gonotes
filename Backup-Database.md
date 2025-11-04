# Backup Database in a Container



Backing up a MySQL database running inside a Docker container typically involves using the `mysqldump` utility and then transferring the backup file out of the container.



## 1. Access the MySQL Container

First, you need to execute commands within your running MySQL container. You can do this using `docker exec`:

```shell
docker exec -it <mysql_container_name_or_id> bash
```

Replace `<mysql_container_name_or_id>` with the actual name or ID of your MySQL container. This command will open an interactive shell session inside the container.



## 2. Create the Backup using `mysqldump`

Once inside the container, use `mysqldump` to create a backup of your database. You will need the MySQL root password or credentials for a user with appropriate backup privileges.

```shell
mysqldump -u root -p'myrootpassword' <database_name> > /tmp/<backup_file_name>.sql
```

- Replace `myrootpassword` with the actual password.
- Replace `<database_name>` with the name of the database you want to back up.
- Replace `<backup_file_name>.sql` with your desired filename for the backup.
- `/tmp/` is a common temporary directory within containers.

#### To back up all databases:

```shell
mysqldump -u root -p<your_mysql_root_password> --all-databases > /tmp/<backup_file_name>.sql
```



## 3. Exit the Container Shell

After the `mysqldump` command completes, you can exit the container's shell:

```
exit
```



## 4. Copy the Backup File to the Host Machine:

Now, copy the backup file from the container to your local machine using `docker cp`:

```
docker cp <mysql_container_name_or_id>:/tmp/<backup_file_name>.sql /path/to/local/backup/directory/
```

- Replace `<mysql_container_name_or_id>` with the container's name or ID.
- Replace `<backup_file_name>.sql` with the name you used for the backup file.
- Replace `/path/to/local/backup/directory/` with the desired path on your host machine where you want to store the backup.



## Alternative: Using Docker Volumes for Persistent Backups

For a more robust and automated backup solution, consider using Docker volumes. You can mount a volume from your host machine into the MySQL container, and then direct `mysqldump` to save the backup directly to that mounted volume. This eliminates the need for the `docker cp` step.



#### Example with Docker Compose:

```
services:
  mysql:
    image: mysql:latest
    container_name: db-ctnr
    environment:
      MYSQL_ROOT_PASSWORD: your_root_password
      MYSQL_DATABASE: myapp-db
    volumes:
      - data:/var/lib/mysql
      - ./backups:/backups # Mount a local directory for backups
  
volumes:
  data:
```

Then, you can execute `mysqldump` within the container, directing the output to the `/backups` directory, which will be directly accessible on your host machine within the `./backups` folder.

```
docker exec -it db-ctnr mysqldump -u root -p'your_root_password' myapp-db > ./backups/myapp-db-20251104.sql
```



## The End