# caddy-l4-docker

A minimal Docker setup for running [Caddy L4](https://github.com/mholt/caddy-l4).

## Usage

1. Clone this repository.
2. Build the Docker image:
	```sh
	docker build -t caddy-l4 -f caddy-l4/Dockerfile caddy-l4
	```
3. Run the container:
	```sh
	docker run --rm -p 443:443 caddy-l4
	```

## Reference
- [Caddy L4 GitHub](https://github.com/mholt/caddy-l4)
