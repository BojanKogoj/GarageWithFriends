# Garage With Friends

Simple docker compose stack for a self-hosted S3 cluster. Contains the following components:

- [Garage](https://garagehq.deuxfleurs.fr/) - an S3-compatible object storage server.
- [Newt](https://github.com/fosrl/newt) - a Pangolin site client.
- [Tailscale](https://tailscale.com/) - a Tailscale client.

Bring your own:

- Pangolin instance, cloud or self-hosted.
- Tailscale coordination server, or Headscale for self-hosted variant.

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

## Services

| Service        | Purpose                                           |
|----------------|---------------------------------------------------|
| `garage`       | S3-compatible object storage.                     |
| `garage-webui` | Admin dashboard for Garage.                       |
| `tailscale`    | For p2p between the nodes via tailnet.            |
| `newt`         | Tunnel client for Pangolin, for public s3 access. |

## Ports

| Port | Purpose                          |
|------|----------------------------------|
| 3900 | S3 API                           |
| 3901 | Node-to-node RPC                 |
| 3902 | Static website hosting           |
| 3903 | Admin API and Prometheus metrics |
| 3909 | Garage admin dashboard           |
