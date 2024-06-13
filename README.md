# Docker compose include environment handling

When the docker compose `include` directive is used there is no way to override environment variables set in the included compose file _unless_ they are already interpolated from the shell in which case the `env_file` option in the long for of the include syntax is useful. The [stated purpose](https://docs.docker.com/compose/compose-file/14-include/#env_file) of this option is

> defines an environment file(s) to use to define default values when interpolating variables in the Compose file being parsed

So it's clear that this option is not designed for injecting an arbitrary set of env vars.

This repo demonstrates one possible workaround method for injecting arbitrary environment _but_ it requires the included project to be modified so it will not help in every case.

The included project can make use of an _optional_ `env_file` and read _that_ path from the environment. In this way we can inject an arbitrary number of environment variables into a container. Handling the file system path to the included env file is the trickiest problem.

## Method

Add this entry to the `env_file` property for the desired container in the included docker compose file

```
- path: "${OVERRIDE_ENV_FILE}"
  required: false
```

Include this docker-compose file in the normal way

```
include:
  - include/docker-compose.yml
```

Create a file `.env.override` and enter any particular environment variables you need in it.

Now we just need to ensure that at the time docker compose runs this variable is present in the env. Note though that the path to `.env.override` is evaluated relative to the included docker-compose file so you'll likely want to set the path absolutely e.g.

```
OVERRIDE_ENV_FILE=$(realpath .env.override) docker compose up
```

or

```
OVERRIDE_ENV_FILE=$PWD/.env.override docker compose up
```

## Beware environment precedence!

Docker compose [environment precedence](https://docs.docker.com/compose/environment-variables/envvars-precedence/) means that `env_file` has lower precedence than the `environment` directive in compose file. Ideally then just don't use the `environment` section. If you really need to override variables set in an `environment` section then they will have to be interpolated from the environment and then you can use the `env_file` directive in the long form of the include syntax.

## Demo

The demo app will exit 0 if an environment variable `$MESSAGE == "included"`. Otherwise it will exit 1.

```
docker compose up --build --exit-code-from envcheck
```

Will exit 1 as the default `MESSAGE` is "default".

```
OVERRIDE_ENV_FILE=$(realpath .env.override) docker compose up --build --exit-code-from envcheck
```

Will exit 0 as the `MESSAGE` is overidden by the included `.env.override`
