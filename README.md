# Garage

A self-hosted, single-node [Garage](https://garagehq.deuxfleurs.fr/) S3-compatible
object storage server, run with Docker Compose.

## Services

| Service     | Purpose                                                                             |
| ----------- | ----------------------------------------------------------------------------------- |
| `garage`    | S3-compatible object storage.                                                       |
| `tailscale` | Joins the host to your Tailscale network for private access.                        |
| `newt`      | Tunnel client for [Pangolin](https://github.com/fosrl/pangolin), for public access. |

All containers run as an unprivileged user (`1000:1000`) with a read-only root
filesystem, all capabilities dropped and `no-new-privileges` set.

## Files

| File           | Contents                                                    |
| -------------- | ----------------------------------------------------------- |
| `compose.yaml` | Service definitions.                                        |
| `garage.toml`  | Garage configuration. Contains no secrets.                  |
| `example.env`  | Template for `.env`, which holds secrets and the data path. |

## Setup

1. Copy the environment template and fill in every value:

   ```sh
   cp example.env .env
   ```

   Generate the Garage secrets with:

   ```sh
   openssl rand -hex 32     # GARAGE_RPC_SECRET
   openssl rand -base64 32  # GARAGE_ADMIN_TOKEN and GARAGE_METRICS_TOKEN
   ```

2. Edit `garage.toml` and replace the `example.com` domains with your own.

3. Create the data directories and give them to the container user:

   ```sh
   mkdir -p "$DATA_ROOT"/garage/meta "$DATA_ROOT"/garage/data "$DATA_ROOT"/tailscale
   sudo chown -R 1000:1000 "$DATA_ROOT"/garage "$DATA_ROOT"/tailscale
   ```

4. Start the stack:

   ```sh
   docker compose up -d
   ```

5. Assign the node a role in the cluster layout. Garage will not store data
   until this is done:

   ```sh
   docker compose exec garage /garage status
   docker compose exec garage /garage layout assign -z dc1 -c 100G <node-id>
   docker compose exec garage /garage layout apply --version 1
   ```

   Use the node ID printed by `status` and set `-c` to the capacity you want
   to give Garage.

6. Create a bucket and an access key:

   ```sh
   docker compose exec garage /garage bucket create my-bucket
   docker compose exec garage /garage key create my-key
   docker compose exec garage /garage bucket allow --read --write my-bucket --key my-key
   ```

## Ports

| Port | Purpose                          |
| ---- | -------------------------------- |
| 3900 | S3 API                           |
| 3901 | Node-to-node RPC                 |
| 3902 | Static website hosting           |
| 3903 | Admin API and Prometheus metrics |

No ports are published on the host. Access goes through Tailscale or Pangolin.

## License

[MIT](LICENSE)
