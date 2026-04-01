# Local XDC (Cross-DC) Setup

Three-cluster Temporal setup on ports `7233`, `8233`, `9233` with a global namespace replicated across all three.

## 1. Start dependencies (CDC)

```sh
make start-dependencies-cdc
```

## 2. Install schema (Postgres XDC)

```sh
make install-schema-postgresql-xdc
```

## 3. Wire up cluster connections (full mesh)

Each cluster needs an upsert pointing at every other cluster's frontend. Six upserts, looped:

```sh
for src in 7233 8233 9233; do
  for dst in 7233 8233 9233; do
    [ "$src" = "$dst" ] && continue
    temporal operator cluster upsert \
      --address 127.0.0.1:$src \
      --frontend-address 127.0.0.1:$dst \
      --enable-connection
      --enable-replication
  done
done
```

## 4. Create the global namespace

```sh
temporal operator namespace create \
  --namespace global-ns \
  --global \
  --active-cluster cluster-a \
  --cluster cluster-a \
  --cluster cluster-b \
  --cluster cluster-c
```

## Verify

```sh
temporal --address 127.0.0.1:7233 operator cluster list
temporal --address 127.0.0.1:7233 operator namespace describe global-ns
```