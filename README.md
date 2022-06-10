# tyk-sync-test

### tyk-sync with tyk-pro-docker-demo

```docker run -it --rm tykio/tyk-sync:v1.2rc3 help```

```
docker run --rm --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp tykio/tyk-sync:v1.2rc3 dump -d="http://localhost:3000/" -s="352d20ee67be67f6340b4c0605b044b7" -t="./tmp"
```
## 1. Dump
docker run --rm --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp \
 tykio/tyk-sync:v1.2rc3 \
 dump \
 -d="http://host.docker.internal:3000" \
 -s="9af9bc28f9dc47d46ad825920820ad3e" \
 -t="./tmp"

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
  --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  sync \
  -d="http://localhost:3000" \
  -s="9af9bc28f9dc47d46ad825920820ad3e" \
  -b="main" git@github.com:LLe27/tyk-sync-test.git
```