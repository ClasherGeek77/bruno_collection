# bruno_collection

## what it is

a Bruno API collection with 29 requests (60 test assertions) for the local reqres.in clone https://github.com/ClasherGeek77/reqresin_clone. requests are plain `.bru` text files, so they diff and review like code. made with Bruno 1.34.1.

layout, under `reqres/`:

| folder | requests | what it covers |
| --- | --- | --- |
| `USERS` | 8 | list, single, not found, create, PUT, PATCH, delete, delayed response |
| `RESOURCES` | 3 | list, single, not found (the color resources) |
| `PRODUCTS` | 1 | list products |
| `AUTH` | 4 | register and login, success and failure |
| `EXTRAS` | 7 | avatar image, unknown filter, `user-list`, edge cases for update, delete, create and register |
| `CHAIN` | 6 | create -> read back -> update -> delete -> read back again, passing a value from one request to the next with `bru.setVar` |

the environment `local` (`environments/local.bru`) holds `base_url = http://localhost:40000` (the reqres clone) and `sd_url = http://localhost:50000` (the saucedemo clone, not used by any request here).

## when to use it

- to learn how a Bruno collection, environment, test script and request chain look.
- to click through the clone's API by hand, or to run all checks from the command line.

when not to use it:

- if your team uses Postman, use https://github.com/ClasherGeek77/postman_collection: the same 29 requests ported to Postman.
- for an automated test suite with reports in a repo, use https://github.com/ClasherGeek77/pytest_api_boilerplate or https://github.com/ClasherGeek77/katalon_api_boilerplate.
- this collection targets the clone, not the live https://reqres.in/: the CHAIN folder and a few checks rely on the clone's behavior (fixed id `4`, stateless).

## what you need

- Bruno desktop app 1.34.1 or newer: https://www.usebruno.com/downloads (the collection was made with 1.34.1; newer versions were not checked).
- the reqres clone, running on port 40000 (below).
- for the command line only: the Bruno CLI `bru`, version 1.34.1 verified (`bru --version` prints `1.34.1`), and node/npm to install it.
- python 3.8 or newer for the clone.

## quick start

1. start the reqres clone and leave it running:

```bash
git clone https://github.com/ClasherGeek77/reqresin_clone.git
cd reqresin_clone
python3 server.py
```

you should see `reqres clone on http://localhost:40000/api  (Ctrl-C to stop)`.

2. get this collection:

```bash
git clone https://github.com/ClasherGeek77/bruno_collection.git
```

3. open it in Bruno: Collections -> Open Collection -> choose the `bruno_collection` folder (the one that contains `bruno.json`). the collection appears as `local-clones`, with one folder `reqres` and the six folders inside.
4. select the environment: top right drop-down, choose `local`. with "No Environment" every request fails because `{{base_url}}` is empty.
5. click `USERS` -> `Single user` -> send. you should see status `200` and a body with `"first_name": "Janet"`, and a green test result.

to run everything in the app: right-click the collection (or the `reqres` folder) -> Run. you should see all requests green. keep the `CHAIN` folder in order: its requests depend on each other.

## run from the command line

from the repo root (the folder with `bruno.json`):

```bash
bru run reqres -r --env local
```

`-r` makes the run recursive into the folders. you should see one line per request with its status, a tick per test, and at the end:

```
Requests:    29 passed, 29 total
Tests:       60 passed, 60 total
```

only one folder:

```bash
bru run reqres/AUTH --env local
```

a different clone port:

```bash
bru run reqres -r --env local --env-var base_url=http://localhost:8000
```

report files: `--reporter-junit out.xml` or `--reporter-html out.html` (options listed by `bru run --help`; not run here).

## how to read the code

- `bruno.json`: marks the folder as a collection.
- `environments/local.bru`: the variables.
- `reqres/<FOLDER>/<name>.bru`: one request each. blocks: `get`/`post`/... (method and url), `body:json`, `script:post-response` (stores values with `bru.setVar`), `tests` (chai `expect`). `meta.seq` is the order of the request.
- `CHAIN`: `Chain 01 plan` sets `plan_name` and `plan_job` in its pre-request script, `Chain 02 create` stores the new id in `new_user_id`, later requests read `{{new_user_id}}`. the clone always answers id `4` and keeps nothing, so `Chain 06` still finds user 4 as Eve: that is the live reqres behavior, on purpose.

## if it breaks

| symptom | cause | fix |
| --- | --- | --- |
| every request fails, error `ECONNREFUSED 127.0.0.1:40000` | the clone is not running | start it: `python3 server.py` in `reqresin_clone` |
| url shows `{{base_url}}/api/...` and fails | no environment selected | pick `local` in the environment drop-down, or pass `--env local` to `bru` |
| `bru: command not found` | CLI not installed | install the Bruno CLI (`npm install -g @usebruno/cli`, not verified here, see https://docs.usebruno.com/) |
| `bru run` finds no requests | you are not in the repo root, or `-r` is missing | `cd` to the folder with `bruno.json` and use `bru run reqres -r --env local` |
| a chain request fails alone | it was run without the requests before it | run the whole `CHAIN` folder |
| `Delayed response` takes about 1 second | it asks the clone for `?delay=1` | expected |

## next

- the same requests for Postman: https://github.com/ClasherGeek77/postman_collection
- the API under test: https://github.com/ClasherGeek77/reqresin_clone
- API tests as code: https://github.com/ClasherGeek77/pytest_api_boilerplate and https://github.com/ClasherGeek77/katalon_api_boilerplate
- load test of the same API: https://github.com/ClasherGeek77/locust_boilerplate
