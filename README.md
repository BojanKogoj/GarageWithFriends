# Garage With Friends

Docker Compose stack for a self-hosted S3 cluster. Contains the following components:

- [Garage](https://garagehq.deuxfleurs.fr/) S3-compatible object storage server.
- [garage-webui](https://github.com/khairul169/garage-webui) admin dashboard.
- [Pangolin](https://github.com/fosrl/newt) site client (newt).
- [Tailscale](https://tailscale.com/) client.

Bring your own:

- Pangolin instance, cloud or self-hosted.
- Tailscale coordination server, or Headscale for self-hosted variant.

## How it works

Connect multiple Garage nodes into a cluster via Tailscale. Tailscale provides network
layer (tailnet), which connects nodes directly to each others, even if servers are behind NAT.

Pangolin exposes the S3 API to the public. Set up a target for each Garage node and with
three or more nodes, you get a scalable highly available self-hosted S3 storage.

## Setup

1. Copy the environment template and fill in every value:

   ```shell
   cp example.env .env
   ```

   Generate the Garage secrets with:

   ```shell
   openssl rand -hex 32     # GARAGE_RPC_SECRET
   openssl rand -base64 32  # GARAGE_ADMIN_TOKEN and GARAGE_METRICS_TOKEN
   ```

2. Edit `garage.toml` and replace the `example.com` domains with your own.

3. Create the data directories and give them to the container user:

   ```shell
   mkdir -p "$DATA_ROOT"/garage/meta "$DATA_ROOT"/garage/data "$DATA_ROOT"/tailscale
   sudo chown -R 1000:1000 "$DATA_ROOT"/garage "$DATA_ROOT"/tailscale
   ```

4. Start the stack:

   ```shell
   docker compose up -d
   ```

5. Gather node IDs and connect the cluster together:

   ```shell
   docker compose exec garage /garage node id
   docker compose exec garage /garage node connect <node-id>
   ```

6. Assign each node a role in the cluster layout. Garage will not store data
   until this is done:

   ```shell
   docker compose exec garage /garage status
   docker compose exec garage /garage layout assign -z dc1 -c 100G <node-id>
   docker compose exec garage /garage layout apply --version 1
   ```

   Use the node ID printed by `status` and set `-c` to the capacity you want
   to give Garage.

7. Create a bucket and an access key:

   ```shell
   docker compose exec garage /garage bucket create my-bucket
   docker compose exec garage /garage key create my-key
   docker compose exec garage /garage bucket allow --read --write my-bucket --key my-key
   ```
