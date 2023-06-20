
## Install

[archwiki](https://wiki.archlinux.org/title/PostgreSQL) :
1. install `paru postgresql`
2. switch to `postgres` user by `sudo -iu postgres`
3. initial configuration `initdb -D /var/lib/postgres/data`
4. start and enable `postgresql.service`
5. create user, same name as Linux username `createuser --interactive`
6. exit to regular user
7. `createdb mydb`
8. access to database `psql -d mydb`

