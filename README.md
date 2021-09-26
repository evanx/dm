# deno-run-worker

Scrips to run Deno workers.

My intended research direction is to enable Redis-driven microservices that are "fun and fast" to develop and test.

- Javascript/Typescript is popular and familiar
- Deno is secure by default
- Redis maps well to programming data structures

I'm an NodeJS professional who believes Redis and Deno are GREAT. I'm exploring how the development and testing of microservices can be simplified by limiting side-effects to Redis.

BTW My career previously revolved around Java and PostgreSQL. I want to write services that integrate Redis to PostgreSQL such that PostgreSQL can be used for bulk persistence, indirectly via Redis, in a decoupled way that is easily mocked out for development and testing.

If we want to accelerate development, we need to reduce cognitive load by de-scoping side effects. I have decided to explore Redis-only microservices, and applying these to the development of chat bots.

## demo

```shell
./clean.sh && ./demo.sh
```

The `demo.sh` shell script will start a worker for the test Redis stream-driven
`deno-date-iso` service: https://github.com/evanx/deno-date-iso/blob/main/worker.ts

This `demo.sh` script will setup the keys in Redis for this worker, e.g. `deno-date-iso:1:h` for worker ID `1.`

```
$ redish deno-date-iso:1:h
repo https://raw.githubusercontent.com/evanx
class deno-date-iso
version v0.0.3
requestStream deno-date-iso:req:x
responseStream deno-date-iso:res:x
consumerId 1
requestLimit 1
denoOptions --inspect=127.0.0.1:9229
```

The `workerUrl` is built from the repo, class and version passed to our runner.

```
const workerUrl =
  (workerVersion === "local"
    ? [workerRepo, workerClass, "worker.ts"]
    : [workerRepo, workerClass, workerVersion, "worker.ts"]).join(
      "/",
    );
```

Note that a `workerVersion` of `local` is used for local development, where the `workerRepo` is a local folder.

This `demo.sh` script will invoke `bootstrap.sh` as follows:

```
    deno run --allow-net=127.0.0.1:6379 --allow-run ./main.ts \
      ${WORKER_REPO} ${WORKER_CLASS} ${WORKER_VERSION} ${workerId} ||
```

Our Deno runner must create the correct options e.g. `--allow-net` as required by our worker:

```
const cmd = [
  "deno",
  "run",
  ...options,
  workerUrl,
  workerKey,
];`
```

Our worker take a Redis hashes key `workerKey` as its sole CLI parameter. It will configure itself via these Redis hashes. The `workerKey` is built as follows:

```
const workerKey = `${workerClass}:${workerId}:h`;
```

Each worker monitors its `pid` field of its hashes, and must exit if this field changes.

We setup Deno options with `--inspect` and can attach a debugger as seen below:

![image](https://user-images.githubusercontent.com/899558/134762517-4ccc28b3-6f8e-4ab9-8529-49054eb7f1ee.png)

<hr>
