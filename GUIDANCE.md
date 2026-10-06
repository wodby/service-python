# Python on Wodby

What Wodby sets up for an application that runs on this service. Check it before adding a server configuration or connection settings to the code.

## How the application is started

The container runs Gunicorn with a configuration file the image writes on every start:

```
gunicorn -c /usr/local/etc/gunicorn/config.py --pythonpath "$GUNICORN_PYTHONPATH" "$GUNICORN_APP"
```

- `GUNICORN_APP` names the WSGI callable as `module:callable`. The image default is `myapp.wsgi:application`; set the variable on the service, or in the Dockerfile, when the application lives elsewhere.
- Gunicorn listens on `0.0.0.0:8080`. The port is fixed in the generated configuration and is the port of the service's HTTP endpoint.
- The image does not include Gunicorn. It must be one of the application's dependencies.
- Access and error logs go to the container's output.

Do not add a Gunicorn configuration file to the repository to change workers or timeouts: the container does not load it. Set these variables on the service instead: `GUNICORN_WORKERS`, `GUNICORN_WORKER_CLASS`, `GUNICORN_WORKER_CONNECTIONS`, `GUNICORN_TIMEOUT`, `GUNICORN_KEEPALIVE`, `GUNICORN_BACKLOG`, `GUNICORN_LOGLEVEL`, `GUNICORN_PROC_NAME`, `GUNICORN_PYTHONPATH`. A change applies with the next deployment of the service.

## Linked services

Links to other services reach the application as environment variables. Nothing in the image reads them: the application reads them itself. Do not hardcode hosts or credentials.

| Link | Variables |
| --- | --- |
| Database (MariaDB, MySQL or PostgreSQL) | `DB_HOST`, `DB_PORT`, `DB_NAME` (also `DB_DATABASE`), `DB_USERNAME`, `DB_PASSWORD`, `DB_DRIVER` (also `DB_CONNECTION`) |
| Mail | `SMTP_HOST`, `SMTP_PORT` |
| Redis or Valkey | `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD` |

All three links are optional. A variable is present only while its link exists and the linked service is enabled. The mail link carries no credentials.

## Environment

- `WODBY_HOSTS` is a JSON array of the environment's hosts, not a comma-separated string. `WODBY_PRIMARY_HOST` and `WODBY_PRIMARY_URL` are the canonical ones for links generated outside a request.
- `WODBY_APP_SERVICE_NAME` is this service's host name inside the environment. Other services reach the application at that name on port 8080.
- `WODBY_ENV_TYPE` tells a development environment from a production-like one.

## Build

The code is copied to `/usr/src/app`, the working directory. The service's own Dockerfile then runs `pip install -r requirements.txt` when that file exists, and nothing else. A pipeline that passes a Dockerfile from the repository (`wodby ci build python -f Dockerfile`) uses that file instead, and the repository's Dockerfile then owns dependency installation. `uv` is available in the image.

The service declares no volume of its own: files written inside the container are lost when it is replaced.

## In a development workspace

- The checkout is mounted at `/usr/src/app` and served as it is on disk.
- Workspace setup installs dependencies with `workspace-python prepare`: `uv sync --locked` when `uv.lock` exists, otherwise `pip install -r requirements.txt` into a virtual environment. One of the two must exist. Run it again after changing dependencies.
- The virtual environment is `.wodby-workspace/venv` in the checkout. `.wodby-workspace/` and `__pycache__/` are kept out of Git status without touching `.gitignore`. Use `.wodby-workspace/venv/bin/python` for commands that need the application's packages.
- The application is started with `python -m gunicorn --bind "$HOST:$PORT" --reload --reload-engine poll "$GUNICORN_APP"`, with `HOST` and `PORT` defaulting to `0.0.0.0` and `8080`. The generated Gunicorn configuration and the `GUNICORN_*` tuning variables are not used here.
- `WORKSPACE_PYTHON_COMMAND` on the service replaces that start command. A custom command must do its own reloading.
- A change to variables or linked services still needs a deployment of the environment.

## Check the result

- `curl -s -o /dev/null -w '%{http_code}' localhost:8080` from the container shows whether the application answers on the expected port.
- `printenv | grep -E '^(DB|SMTP|REDIS)_' | cut -d= -f1` lists the link variables that are present.
