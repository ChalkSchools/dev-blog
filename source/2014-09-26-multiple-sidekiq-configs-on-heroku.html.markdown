---
title: Multiple Sidekiq Configs on Heroku
date: 2014-09-26
tags: sidekiq, heroku
authors: Holman Gao
---

## TL;DR

If you need to use two different Sidekiq configs on Heroku, it is super easy!
In your Heroku `Procfile`, just specify:

```yaml
# Procfile
web: bundle exec unicorn -p $PORT -c ./config/unicorn.rb
worker_1: bundle exec sidekiq -C config/sidekiq_1.yml
worker_2: bundle exec sidekiq -C config/sidekiq_2.yml
```

## Problem

We heavily use Sidekiq for asynchronous jobs at Chalk, where we create jobs for
tasks such as uploading files, sending emails, and processing documents.  We
ran into an issue where we had several jobs that were hogging a ton of memory,
and we needed to reduce the concurrency to keep those jobs from crashing.
However, we also wanted to keep the concurrency high so that low intensity jobs
(like sending out emails) could still complete quickly.

Our app is deployed to Heroku.

## Solution

We needed to create an additional worker, dedicated to handling the memory
intensive (and slow) jobs.  This way, the slow jobs can just chug along, while
the normal jobs run on their own, uninterfered.  Luckily, it is actually
relatively simple to handle this case through Sidekiq and Heroku.

1. Sidekiq supports different queues, so in addition to the `:default` queue,
   we added a `:slow` queue.  We send our high memory jobs to the `:slow`
   queue, by doing:

    ```ruby
    # slow_worker.rb
    class SlowWorker
      include Sidekiq::Worker
      sidekiq_options queue: :slow
      ...
    ```
  <br />

2. We created an additional Sidekiq config, where we specify it to only handle
   jobs from the `:slow` queue.  By default, Sidekiq works from the `:default`
   queue, so we did not to modify our original config.

    ```yaml
    # config/sidekiq.yml
    :concurrency: 20

    # config/sidekiq_slow.yml
    :concurrency: 1
    :queues:
      slow
    ```
  <br />

3. We updated our `Procfile` to let Heroku know to create two different types
   of workers.

    ```yaml
    # Procfile
    web: bundle exec unicorn -p $PORT -c ./config/unicorn.rb
    worker: bundle exec sidekiq -C config/sidekiq.yml
    slow_worker: bundle exec sidekiq -C config/sidekiq_slow.yml
    ```
  <br />

4. Push to Heroku, and scale `worker` and `slow_worker` processes as
   appropriate (the workers can be scaled on the Heroku website through the
   admin panel, or on the command line with `heroku ps:scale slow_worker=2`).
   This is awesome because we can use a different number of `worker`s and
   `slow_worker`s, depending on our needs!  Well done, Sidekiq and Heroku,
   for making this setup so easy!
