# tyk-sync-test
## Prerequisites
- Tyk-Sync was built using Go 1.10. The minimum Go version required to install is 1.7.
- In order for policy ID matching to work correctly, your Dashboard must have `allow_explicit_policy_id: true` and `enable_duplicate_slugs: true` and your - Gateway must have `policies.allow_explicit_policy_id: true`.
- It is assumed you have a Tyk CE or Tyk Pro installation.

> After migrating a Policy from one environment to another, it is important to note that the displayed Policy ID is not going to match. That is okay. It happens because Tyk Dashboard displays the `Mongo ObjectId`, which is the `_id` field, but the id is the important part.

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
 -s="{API_SECRET_FROM_USER_PROFILE}" \
 -t="./tmp"

cd ..
```

### 3a. `publish` - Publish API definitions from a Git repo to a Tyk Gateway or Dashboard.
```
docker run --rm \
  --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  publish \
  -d="http://host.docker.internal:3000" \
  -s="{API_SECRET_FROM_USER_PROFILE}" \
  -p="./tmp" 
```

### 3b. `sync` - Synchronise an API Gateway with the contents of a Github repository.
- One cross-platform solution is to use a bind mount to share the host's .ssh folder to the container.
- One caveat to consider, however, is that all contents (including private keys) from the .ssh folder will be shared so this approach is only desirable for development and only for trusted container images.
```
docker run --rm \
  --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp \
  -v ~/.ssh:/root/.ssh \
  tykio/tyk-sync:v1.2rc3 \
  sync \
  -d="http://host.docker.internal:3000" \
  -s="{API_SECRET_FROM_USER_PROFILE}" \
  -k="/opt/tyk-sync/tmp/key/tyk_sync_key" \
  -b="refs/heads/{REPOSITORY_BRANCH_NAME}" git@github.com:{USERNAME}/{REPOSITORY_NAME}.git
```

### 3c. `update` - Attempt to identify matching APIs or Policies in the target, and update those APIs. It does not create new ones.
```
docker run --rm \
  --mount type=bind,source="$(pwd)",target=/opt/tyk-sync/tmp \
  tykio/tyk-sync:v1.2rc3 \
  update \
  -d="http://host.docker.internal:3000" \
  -s="{API_SECRET_FROM_USER_PROFILE}" \
  -p="./tmp" \
  -k="/opt/tyk-sync/key/tyk_sync_key"
```

#### Note:
But what about upper environments (pre-prod/prod) ? The concern here is that each Tyk env will be connected to different backend systems, so the target URL´s will change among other stuff (virtual endpoints/middleware configs/etc), I assume this must be a manual change after deploy and subsequent dump of the upper-envs to back-up our configs as code. 

A script will need to be created in order to change the manual