# How to use this image

### Create a `Dockerfile` in your PHP project

```dockerfile
FROM nmoeini/php:7.1-bionic
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
CMD [ "php", "./your-script.php" ]
```

Then, run the commands to build and run the Docker image:

```console
$ docker build -t my-php-app .
$ docker run -it --rm --name my-running-app my-php-app
```

## Running as an arbitrary user

For running the FPM variants as an arbitrary user, the `--user` flag to `docker run` should be used (which can accept both a username/group in the container's `/etc/passwd` file like `--user daemon` or a specific UID/GID like `--user 1000:1000`).

## Extensions

In many production environments, it is also recommended to (build and) enable the PHP core OPcache extension for performance. See [the upstream OPcache documentation](https://www.php.net/manual/en/book.opcache.php) for more details.

In development environments, it is also recommended to (build and) enable the XDebug extension. See [the Xdebug documentation](https://xdebug.org/docs/) for more details. 

## With a `Dockerfile`

```dockerfile
FROM nmoeini/php:7.1-bionic
COPY src/ /var/www/html/
```

Where `src/` is the directory containing all your PHP code. Then, run the commands to build and run the Docker image:

```console
$ docker build -t my-php-app .
$ docker run -d --name my-running-app my-php-app
```

## Without a `Dockerfile`

```console
$ docker run -d -p 80:80 --name my-apache-php-app -v "$PWD":/var/www/html nmoeini/php:7.1-bionic
```