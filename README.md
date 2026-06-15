# jovay-docker

Docker Compose for a self-hosted [Jovay](https://docs.jovay.io/guide/jovay-rpc-node) L2 RPC node.

## Setup

```bash
cp default.env .env
nano .env  # set DOMAIN, RPC_HOST, WS_HOST, SHARE_IP
```

## First Run

The config files are embedded in the Docker image — extract them before starting:

```bash
IMAGE="jovay-release-registry.cn-hongkong.cr.aliyuncs.com/jovay/l2-rpc:0.13.0-rc1"
docker create --name temp_container ${IMAGE}
docker cp temp_container:/opt/l2_deploy/conf ./conf
docker rm temp_container
```

Download genesis and VERSION into `conf/`:

```bash
wget http://dl.jovay.io/snapshot/genesis.conf -O genesis.conf
echo "502c910cbc21137c606621622fe67d28 genesis.conf" | md5sum -c -
mv genesis.conf ./conf/genesis.aldaba-ng.conf

wget http://dl.jovay.io/snapshot/VERSION_epoch16905 -O VERSION
echo "3b8bfc464d24c101e0b407aeba200345 VERSION" | md5sum -c -
mv VERSION ./conf/VERSION
```

> `genesis.conf` must be renamed to `genesis.aldaba-ng.conf`

Start the node and wait for it to sync a few blocks before proceeding:

```bash
./jovayd up
./jovayd logs -f jovay
```

## Snapshot Restore

Once the node has synced a few blocks, stop it and restore from snapshot:

```bash
./jovayd down

wget -c http://dl.jovay.io/snapshot/20260508_8477395.tar.gz
echo "482095d707ba3e41a4e7fc8b3c14e4bf 20260508_8477395.tar.gz" | md5sum -c -

docker run --rm -v jovay-docker_jovay-data:/data alpine sh -c \
  "mv /data/public /data/public.backup && rm -rf /data/history_kvdbs"
tar -zxvf 20260508_8477395.tar.gz -C /var/lib/docker/volumes/jovay-docker_jovay-data/_data/
docker run --rm -v jovay-docker_jovay-data:/data alpine chown -R 1000:1000 /data

./jovayd up
```

## Verify

```bash
curl 127.0.0.1:18100/ \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{"method":"eth_blockNumber","params":[],"id":1,"jsonrpc":"2.0"}'
```

## License

[Apache License 2.0](LICENSE)
