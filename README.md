wp-SITES
---

This project is structured to spin up as many docker wordpress instances as needed on a single host. 
This project is served by nginx. Ideally, with nginx-proxy so that each site is configured with nginx as each site is brought online. So nginx-proxy container must be started before site containers. To do so, run the following:
- `docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro –-restart=always jwilder/nginx-proxy`

Note: To use letsencrypt for https, this command is modified to work with nginx-proxy-companion. This stores the site ssl certs in the currently directiory/certs. The command for this is:
- `docker run -d --name=companion -e "DEBUG=true" -v $(pwd)/certs/:/etc/nginx/certs:rw --volumes-from nginx-reverse-proxy -v /var/run/docker.sock:/var/run/docker.sock:ro jrcs/letsencrypt-nginx-proxy-companion`

Note2: To view the companion logs, run:
- `docker logs -f companion`

The project structure is as follows:

- `./sites.yml`
	This is the docker compose file. It contains container descriptions for mysql and each wordpress site.
- `./mysql/`
	This is the directory containing all mysql data for the websites. This can be backed up as an entire directory, but the easier way to create a backup of your data is to generate a sql dump.
- `./certs/`
	This is the directory containing all the certs for the nginx-letsencrypt-companion. (optional)
- `./www/`
	This is the directory containing each websites in its own directory. This can be backed up as an entire directory for your sites, but the more sane approach is to backup only what you need such as: plugins, images, theme, posts, pages, etc.

To start the sites, run: 
- `docker-compose -f sites.yml up -d --no-recreate`

If there are any problems and you want to restart everything, run:
- `docker rm -f companion; docker-compose -f sites.yml down; docker run -d --name=companion -e "DEBUG=true" -v /root/certs/:/etc/nginx/certs:rw --volumes-from nginx-reverse-proxy -v /var/run/docker.sock:/var/run/docker.sock:ro jrcs/letsencrypt-nginx-proxy-companion ; docker-compose -f sites.yml up -d --no-recreate; docker logs -f companion`

To backup mysql, run this command:
- `docker exec some-mariadb sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql`

To restore a backup of mysql, run this command:
- `docker exec -i my-new-database mysql -uroot -p"MYSQL_ROOT_PASSWORD" --force < all-database.sql`
