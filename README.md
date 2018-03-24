# docker_django
Django hello_world running on gunicorn + caddy

## Showing Hello World
Clone the project to your machine

```
git clone https://github.com/jjorissen52/docker_django.git
```

Configure `/hello_world/caddy/docker-compose.yml` so that the `WORKER` environment variable refers
to the ip address where your gunicorn worker lives

```
    environment:
          - WORKER=192.168.50.11 # ip where my gunicorn worker(s) resides
```

```

cd docker_django/hello_world
sudo docker-compose build # build our django container
sudo docker-compose up # get our django container which will be running gunicorn on port 8000
# sudo docker-compose up -d # flag to run silently
# sudo docker-compose down # to take down any silent containers described by docker-compose.yml in current directory
cd caddy
sudo docker-compose up
```
That's it! Point yourself to localhost and you should see the Django Hello World page!

## Caddy Image
This guy has made it available on many different CPU architectures. 

https://github.com/marcbachmann/caddy

## Caddy Config `/hello_world/caddy/docker-compose.yml`
```
version: '3'
services:
  caddy:
    image: marcbachmann/caddy:0.10.7-arm7-semi # change arm7 to amd64 or whatever you need
    container_name: caddy-arm
    volumes:
      - ./config:/config # image needs access to Caddyfile in /config/
      - ./static:/static # mounting a static folder to be accessed by caddy (not in project)
    ports:
      - "80:80" # both ports need to be forwarded to the outside world for TLS certificate to be granted
      - "443:443"
    environment:
      - WORKER=192.168.50.11 # ip where my gunicorn worker(s) reside
    command: -conf /config/Caddyfile #it needs this flag to know where the server config will be on its filesystem
```

## Caddy Config `/hello_world/caddy/config/Caddyfile`
```
# Create an A record pointing to a public IP that can reach your 
# server and you will be granted a cert. That's all it takes.
your.domain.com {
    # root is needed to serve static files; if you
    # are using cloudfront or some other method of
    # serving static files you can comment out root
    root /static 
    proxy / http://{$WORKER}:8000 {
        transparent
    }
}
```
