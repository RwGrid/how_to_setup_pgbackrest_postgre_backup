# how_to_setup_pgbackrest_postgre_backup
Complete configuration documentation of pgbackrest with postgres and pg_dump
ok
First things first 
Follow this link to install Postgres on ubuntu:
https://www.postgresql.org/download/linux/ubuntu/

Second:
Check health of postgres using following tutorial 

https://www.commandprompt.com/education/how-to-check-postgresql-service-status-in-linuxubuntu/

Third: must setup password of the postgres user 

1. Log in as the system user that PostgreSQL is running as (usually "postgres"). You can typically switch to this user using the `su` command on Linux or macOS:

   ```
   su - postgres
   ```

2. Once you are the "postgres" user, you can connect to the PostgreSQL database server:

   ```
   psql
   ```

3. Inside the PostgreSQL shell, you can set a password for the "postgres" user or any other user using SQL commands. To change the "postgres" user's password, you can use the following SQL command:

   ```
   ALTER USER postgres PASSWORD 'new_password';
   ```

   Replace `'new_password'` with the password you want to set.

4. Exit the PostgreSQL shell by typing:

   ```
   \q
   ```

5. You can then exit the "postgres" user session by typing:

   ```
   exit
   ```
Fourth: If there is a problem in getting health in step two
If you're not getting any output when you run the `sudo systemctl status postgresql` command, it means that the systemctl service for PostgreSQL is not running or may not be properly installed or configured on your system. Here are some steps you can take to troubleshoot and resolve the issue:

1. **Check PostgreSQL Installation:**
   Ensure that PostgreSQL is installed on your system. You can do this by running the following command:
   ```
   sudo apt list --installed | grep postgres
   ```
   If PostgreSQL is not installed, you will need to install it using your package manager. For example, on Debian-based systems, you can use:
   ```
   sudo apt-get install postgresql
   ```

2. **Start PostgreSQL Service:**
   If PostgreSQL is installed but not running, you can start the service using the following command:
   ```
   sudo systemctl start postgresql
   ```

3. **Enable PostgreSQL Service:**
   To ensure that PostgreSQL starts automatically when your system boots, enable the service with the following command:
   ```
   sudo systemctl enable postgresql
   ```

4. **Check PostgreSQL Service Status:**
   After starting PostgreSQL, check its status again with:
   ```
   sudo systemctl status postgresql
   ```
--------note--------- Pgbackrest Perform backups on a cluster and not a singular database ---
Fifth: FOLLOW THIS Tutorial to the letter https://pgbackrest.org/user-guide.html

this is the conf file in /etc/pgbackrest/pgbackrest.conf

step: modify settings in postgres to allow WAL configration :

sudo nano /etc/postgresql/15/main/postgresql.conf 

must add these : 

archive_command = 'pgbackrest --stanza=main archive-push %p'
archive_mode = on
max_wal_senders = 3
wal_level = replica

then i must restart cluster 

sudo pg_ctlcluster 15 main  restart

step: create conf file of pgbackrest
---------------------------------------------------------
[global]
repo1-path=/var/lib/pgbackrest
log-level-console=info
log-level-file = debug
repo1-retention-full=2
repo1-cipher-pass=zWaf6XtpjIVZC5444yXB+cgFDFl7MxGlgkZSaoPvTGirhPygu4jOKOXf9LO4vjfO
repo1-cipher-type=aes-256-cbc
[main]
pg1-path=/var/lib/postgresql/15/main
db-port=5432
[backup]
repo1-retention-full=7 # Keep full backups for 7 days (adjust as needed)
repo1-retention-diff=35 # Keep differential backups for 35 days (adjust as needed)
start-fast=y
[global:archive-push]
compress-level=3
-----------------------------------------------------

If u notice here , iam using postgres user to perform the stanza 

the password of postgres is defined  in third step

766  sudo -u postgres pgbackrest --stanza=main --log-level-console=info stanza-create
  
767  sudo -u postgres pgbackrest --stanza=main --log-level-console=info check

sudo -u postgres pgbackrest --stanza=main \
       --log-level-console=info backup

<h2>Cron job :</h2>

crontab -e 

add the following 
30 06  *   *   0     pgbackrest --type=full --stanza=main backup
30 06  *   *   1-6   pgbackrest --type=diff --stanza=main backup

perform cron job 

backups are scheduled for 6:30 AM every Sunday with differential backups scheduled for 6:30 AM Monday through Saturday.

Get backup information about backups that are done using pback_rest

sudo -u postgres pgbackrest info

<h2>TO configure pgadmin with this: </h2>

step1 : Must first allow hosts to access postgres from other ips such as container of pgadmin

cd etc/postgresql/15/main
nano postgresql.conf 
change listen_address=localhost to  ---->listen_addresses = '*' 

step 2:
 sudo nano /etc/postgresql/15/main/pg_hba.conf
 add this to end of file
 host    all             all             0.0.0.0/0               md5
step 3 :
 restart container
 sudo service postgresql restart

additional might use :
to go to postgres user in command line :   use this ---> sudo -i -u postgres
<h3>Restoring from pgbackrest</h3>
https://pgbackrest.org/user-guide.html
<h2>to use terminal base pg_dump</h2>
must use terminal and not in psql 
then follow this tutorial
https://www.youtube.com/watch?v=nbnRibO5Qmo
