# Concourse CI Pipeline Test

This directory contains two Concourse CI simple test pipelines, `hawkscan-remote.yml`, and `hawkscan-local.yml`. They each require two secrets, `hawk_api_key`, and `ssh_private_key`.

**NOTE:** This test uses the `concourse` branch of this repository! Be sure to push any changes you want to see to that branch.

## Test Setup

You can use a simple [Docker Compose](#docker-compose) test environment, or a more complete [BUCC](#bucc) environment to test the Concourse pipelines in this directory.

### Docker Compose

For a simple test of these pipelines, you can use the `docker-compose-concourse-environment.yml` configuration found in this directory to stand up Concourse and a Postgres DB. This has no credential store, so you can provide secrets as parameters when you set up the pipelines from the `fly` CLI.

Here is a sample run that should get you going. This walkthrough assumes your Git SSH key is saved at `~/.ssh/id_rsa`, and your StackHawk API key is save as an environment variable `HAWK_API_KEY`.

```shell
zconger@rey concourse % docker-compose --file docker-compose-concourse-environment.yml up --detach 
```

Now browse to http://127.0.0.1:8080/ to install the Concourse CLI, `fly`. Then:

```shell
fly --target kaakaw login --concourse-url http://127.0.0.1:8080 -u admin -p admin
fly --target kaakaw set-pipeline --pipeline hawkscan-remote --config hawkscan-remote.yml --var hawk_api_key="${HAWK_API_KEY}" --var ssh_private_key="$(cat ~/.ssh/id_rsa)"
fly --target kaakaw set-pipeline --pipeline hawkscan-local --config hawkscan-local.yml --var hawk_api_key="${HAWK_API_KEY}" --var ssh_private_key="$(cat ~/.ssh/id_rsa)"
```

Now log on to the web console at http://127.0.0.1:8080/ and login with username `admin` and password `admin`.

To shut the environment down:

```shell
docker-compose down
```

### BUCC

For a more thorough test using CredHub for credential management, clone the [starkandwayne/bucc](https://github.com/starkandwayne/bucc) repo and bootstrap a BUCC (BOSH, UAA, CredHub, and Concourse CI) stack.

#### Prereqs

Install BOSH and its dependencies on your workstation:
https://bosh.io/docs/cli-v2-install/

Install VirtualBox version 6.latest (later versions won't work):
https://www.virtualbox.org/wiki/Download_Old_Builds_6_0

#### Bootstrap

To bootstrap the BUCC environment, clone the bucc repo, and run the following commands.

```shell
source .envrc
bucc up --lite
bucc fly        # target and log in to Concourse
bucc credhub    # target and log in to CredHub
credhub set -n /concourse/main/ssh_private_key --type value --value "$(cat ~/.ssh/id_rsa)"
credhub set -n /concourse/main/hawk_api_key --type value --value "${HAWK_API_KEY}"
```

Add your credentials to the CredHub service.

```shell
credhub set -n /concourse/main/ssh_private_key --type value --value "$(cat ~/.ssh/id_rsa)"
credhub set -n /concourse/main/hawk_api_key --type value --value "${HAWK_API_KEY}"
```

Then come back to this directory with your BUCC environment still sourced, and set up your pipelines.

```shell
fly --target bucc set-pipeline --pipeline hawkscan-remote --config hawkscan-remote.yml
fly --target bucc set-pipeline --pipeline hawkscan-local --config hawkscan-local.yml
```

Now you can log on to the Concourse web interface to see your new pipelines. Check `bucc info` for login information. For example:

```shell
zconger@rey concourse % bucc info
Concourse:
  url: https://192.168.50.6
  username: admin
  password: yostmf7c6fijyrtvqfpi
```

To shut the environment down:

```shell
bucc down
```
