## Docker PHP development stack

Built for universal PHP development with a focus on Laravel & WordPress tools.

**Not for use in a production environment**

## Currently supports / includes:

- PHP:
  - 8.3 (Imagick not currently supported)
  - 8.2
  - 7.4
- xDebug
- NGINX
- Mysql
- Redis
- NodeJS & Yarn *via NVM*
- Heroku CLI
- Oh-My-Zsh
- AWS cli

## Getting started:

Ensure you have Docker and Docker Compose installed on your system.

Copy `.env.sample` to a new file called `.env`. From this document you can configure various settings including:

- Node & Composer versions
- Logging directories
- Ports
- User's name and IDs within the containers
- Git `--config` details for commits

You dont have to make any changes, as the defaults are all you need to get started. But the `.env` file must exist.

You can then start the Docker Compose suite in detached mode:

```bash
docker compose up -d
```

This will build all of the containers. For the first build, it can occassionally throw an error which is caused from updating the underlying Debian system. Just restart the build if you encounter this on the first try.

While the initial build can take 10+ minutes, once the containers are built you wont have to build them again unless you wish to make changes or fetch updates. I'd recommend doing this once a month if you're interested in keeping up-to-date.

## Hosting a website for development

To host your first website locally, go to the `nginx` directory and create a copy of `nginx/template.conf`. Insert your copy into the `nginx/sites-enabled` folder, being sure to rename it to something appropriate.

Now you should have a file: `nginx/sites-enabled/mywebsite.conf`.

Inside that file, you'll now need to make two changes (indicated by 'MYWEBSITE'), and to choose a PHP version:

```conf
server {
    listen 80;
    server_name MYWEBSITE.localhost; # The subdomain you'd like the website use

    # The directory of the root folder for the website within the container.
    # `/public` is added for this example but remove if needed.
    root /var/www/html/MYWEBSITE/public;
    index index.php index.html;

    # ... REST OF FILE

    # Choose PHP version
    # COMMENT OUT THE ONE YOU DON'T WANT
    #include /etc/nginx/php/7.4.conf;
    include /etc/nginx/php/8.2.conf;
}
```

**In order for the subdomains to work, be sure to add `*.localhost 127.0.0.1` to your host system's `hosts` file.**

Now you've set up the nginx configuration file for the website, just restart the nginx container (or the whole compose setup) and you're good to go!

### To sum up:

- Copy the `nginx/template.conf` file into the `nginx/sites-enabled` folder.
- Rename it to something appropriate.
- Set the `server_name` and `root` directory within the file.
- Restart the `nginx` container.

## Aliases

You can add aliases to be passed to the PHP containers for use within them.

To do so, edit the `php/aliases` file.

## Development with this setup

Now that you've built the containers, added your config file for the website you're building, and added any aliases you're wanting to use, you're ready to code.

Here's my recommendation:

Using VSCode, [install the Docker extension for remote container development](https://code.visualstudio.com/docs/devcontainers/containers). Using this, you can then attach VSCode to a container.

If you attach to one of the PHP containers, for example, you'll then have access to a full Linux environment.

You can use ZSH for a nice terminal experience, run any PHP or Node/NPM commands you'd like, and manage your code via Git.

## Important information

- ZSH is used by default when connecting via VSCode's remote containers extension. If you connect to a terminal session directly via your host terminal or Docker Desktop, it won't use ZSH by default.
- A DNSMasq container is used to allow the PHP containers to access the Nginx servers via their domain names. Leave this enabled and just leave it alone if you're not sure what it does as it's very important.
- When setting up a database connection from a PHP container to the MYSQL container, you must set the IP address of the server to be `mysql`, as this is how Docker Compose containers 'see' eachother, instead of using `localhost` or `127.0.0.1` as you might normally expect.
- If you add SSH keys, these will be saved in the `ssh` folder so that you need only set them up once.
- All data for mysql and PHP containers are stored in Docker volumes. This ensures maximum speed but doesn't allow you to navigate them with your host computer's file browser. For Windows systems, this is to ensure there's no slowdown. If you don't like this, just change the mappings in the `docker-compose.yml` file.

## To do:

- Investigate intermitent issue with first builds crashing during updates.