# tyk-sync-test

### tyk-sync with tyk-pro-docker-demo

```docker run -it --rm tykio/tyk-sync:v1.2rc3 help```

```
docker run --rm --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp tykio/tyk-sync:v1.2rc3 dump -d="http://localhost:3000/" -s="352d20ee67be67f6340b4c0605b044b7" -t="./tmp"
```
## 1. Dump
```
docker run --rm --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp \
 tykio/tyk-sync:v1.2rc3 \
 dump \
 -d="http://host.docker.internal:3000" \
 -s="9af9bc28f9dc47d46ad825920820ad3e" \
 -t="./tmp"
```

## 3. Push changes to Git repo
```
cd tmp
git add .
git commit -m "My dashboard dump"
git push -u origin my-test-branch
```

##  2. Sync using SSH
```
docker run --rm \
  --network="host" \
  --mount type=bind,source="$(pwd)/tmp",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  sync \
  -d="http://localhost:3000" \
  -s="9af9bc28f9dc47d46ad825920820ad3e" \
  -b="refs/heads/my-test-branch" git@github.com:LLe27/tyk-sync-test.git \
  -p="./tmp" \
  -k="/opt/tyk-sync/tmp/key/tyk_sync_key"

docker run --rm \
  --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 sync \
  -d="http://localhost:3000" \
  -s="9af9bc28f9dc47d46ad825920820ad3e" \
  -b="refs/heads/main" git@github.com:LLe27/tyk-sync-test.git \
  -k="./opt/tyk-sync/tmp/key/tyk_sync_key"
```

docker run -it --network="host" --rm --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp tykio/tyk-sync:v1.2rc3 publish -d="http://localhost:3000" -s="9af9bc28f9dc47d46ad825920820ad3e"

docker run --rm \
  --network="host" \
  --mount type=bind,source="$(pwd)/tmp",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  sync \
  -d="http://localhost:3000" \
  -s="a4d3d2cc47924deb638f8928c5682041" \
  -p="./tmp" \
  -k="/opt/tyk-sync/tmp/key/tyk_sync_key"


docker run --rm \
  --network="host" \
  --mount type=bind,source="$(pwd)/tmp",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  publish \
  -d="http://localhost:3000" \
  -s="a4d3d2cc47924deb638f8928c5682041" \
  -p="./tmp" \
  -k="/opt/tyk-sync/tmp/key/tyk_sync_key"

docker run --rm \
  --network="host" \
  --mount type=bind,source="$(pwd)/tmp",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  update \
  -d="http://localhost:3000" \
  -s="a4d3d2cc47924deb638f8928c5682041" \
  -p="./tmp" \
  -k="/opt/tyk-sync/tmp/key/tyk_sync_key"





# tyk-sync-test

## Testing the CI/CD flow
### 1. Create tmp and key directories
```
mkdir tmp
mkdir key
```

### 2. `dump` - Extract policies and APIs from a target (dashboard) and place them in the tmp directory.
```
cd tmp

docker run --rm --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp \
 tykio/tyk-sync:v1.2rc3 \
 dump \
 -d="http://host.docker.internal:3000" \
 -s="a4d3d2cc47924deb638f8928c5682041" \
 -t="./tmp"

cd ..
```

### 3a. `publish` - Publish API definitions from a Git repo to a Tyk Gateway or Dashboard.
```
docker run --rm \
  --network="host" \
  --mount type=bind,source="$(pwd)/tmp",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  publish \
  -d="http://localhost:3000" \
  -s="a4d3d2cc47924deb638f8928c5682041" \
  -p="./tmp" \
  -k="/opt/tyk-sync/key/tyk_sync_key"
```

### 3b. `sync` - Synchronise an API Gateway with the contents of a Github repository.
```
docker run --rm \
  --network="host" \
  --mount type=bind,source="$(pwd)/tmp",target=/opt/tyk-sync/tmp \
  -e SSH_KNOWN_HOSTS="$(< ~/.ssh/known_hosts) \
  tykio/tyk-sync:v1.2rc3 \
  sync \
  -d="http://localhost:3000" \
  -s="a4d3d2cc47924deb638f8928c5682041" \
  -p="./tmp" \
  -k="/opt/tyk-sync/key/tyk_sync_key"
```

docker run --rm \
  --mount type=bind,source="$(pwd)/tmp",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  sync \
  -d="http://localhost:3010" \
  -s="a4d3d2cc47924deb638f8928c5682041" \
  -k="/opt/tyk-sync/tmp/key/tyk_sync_key" \
  -b="refs/heads/my-test-branch" git@github.com:LLe27/tyk-sync-test.git

### 3c. `update` - Attempt to identify matching APIs or Policies in the target, and update those APIs. It does not create new ones.
```
docker run --rm \
  --network="host" \
  --mount type=bind,source="$(pwd)/tmp",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  update \
  -d="http://localhost:3000" \
  -s="a4d3d2cc47924deb638f8928c5682041" \
  -p="./tmp" \
  -k="/opt/tyk-sync/key/tyk_sync_key"
```