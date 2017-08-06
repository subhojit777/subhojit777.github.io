---
title: Run Nightwatch.js tests parallelly in Docker
description: Docker setup to run Nightwatch.js tests parallelly
date: '2017-05-23'
author: Subhojit Paul
#tags:
# - "docker"
# - "nightwatch"
# - "DevOps"
# - "CI"
---

[Nightwatch.js](http://nightwatchjs.org) is a tool used for automated browser,
it is built in Node.js. Recently I had setup GitLab CI which will run
Nightwatch.js tests on every push to `master` branch.

It is very common that the acceptance tests are slow. Therefore, after setting
up the CI, my next requirement was to run the tests parallelly. This was
challenging, given that I am very much new to Docker. Finally, I figured it
out.

### How to run Nightwatch.js in parallel mode
You can either execute the tests in parallel by `test_workers` setting. Or you
can specify multiple environments while running the tests. Read more about
running tests in the parallel mode [here](https://github.com/nightwatchjs/nightwatch-docs/blob/master/guide/running-tests/run-parallel.md).

### How parallel mode in Nightwatch.js works
The basic idea is - it will create a child process for every test. Every child
process is assigned a browser instance. The parent process is the process that
executes the main `nightwatch` command. The parent process will wait for the
child processes (or tests) to complete, and in the end it will show you the
complete output of all the child processes.

--------------------

Earlier I was using
[`subhojit777/nightwatch`](https://github.com/subhojit777/nightwatch) setup to
run the tests. It was working alright until the requirement of parallel
running came. The setup was not able to spawn multiple instances of the browser,
hence the tests were completing within a few seconds without showing any output.

I wrote a [Docker Compose](https://docs.docker.com/compose/overview) script that
uses official Selenium images to configure a Selenium Grid and connect it to
Chrome browser that is going to run the tests.

### The `docker-compose.yml`:

```yml
version: '3'
services:
  hub:
    image: selenium/hub:latest
    ports:
      - 4444:4444
  chromedriver:
    image: selenium/standalone-chrome:latest
    environment:
      HUB_PORT_4444_TCP_ADDR: hub
      HUB_PORT_4444_TCP_PORT: 4444
    depends_on:
      - hub
  nightwatch:
    image: subhojit777/nightwatch:latest
    environment:
      - WAIT_FOR_HOSTS=chromedriver:4444
    depends_on:
      - hub
      - chromedriver
    links:
      - chromedriver
    volumes:
      - ./tests:/home/node
```

### Sample `nightwatch.json`:

```json
{
  "src_folders" : ["tests"],
  "output_folder" : "reports",
  "test_settings" : {
    "default" : {
      "launch_url" : "http://site.local",
      "selenium_port"  : 4444,
      "selenium_host"  : "chromedriver",
      "screenshots" : {
        "enabled" : true,
        "on_failure" : true,
        "on_error" : false,
        "path" : "screenshots"
      },
      "desiredCapabilities": {
        "browserName": "chrome",
      }
    },
    "prod" : {
      "launch_url" : "https://www.myprodsite.com",
      "filter" : "tests/*-prod.js",
    },
    "stg" : {
      "launch_url" : "https://www.mystgsite.com",
      "filter" : "tests/*-stg.js",
    },
    "dev" : {
      "launch_url" : "https://www.mydevsite.com",
      "filter" : "tests/*-dev.js",
    }
  }
}
```

I am assuming that the Nightwatch.js tests of your application are inside
`tests` directory.

Just place the `docker-compose.yml` file inside the docroot of your application.
Make sure the tests are inside `tests` directory, and the tests directory should
be in docroot. Otherwise, you can change the `volumes` entry as per your setup.

Let's assume that you have placed `docker-compose.yml` in the correct location
and the tests are in the right place, execute this command from your application
docroot:

```sh
docker-compose run --rm nightwatch --env prod,stg,dev
```

And the setup will execute production, staging and dev tests parallelly. Say,
dev takes the most time 10 minutes to complete the test, so the process will run
for 10 minutes, and after that it will show you the result of the tests as
output.

And don't forget to clean up.

```sh
docker-compose down -v
```
