# Why Ruby app deploy is getting progressively slower

I have a big and stable Rails application which has been running in production for 5 years. Each time I deploy the code using Capistrano and I've noticed that deploying time is getting slower. It takes 5 minutes longer than we are used to. Looking through the logs, I've noticed that the task `bundle exec rake db:migrate` takes 5 minutes. It was unusual!

I've done some manipulations with a deploy script. I noticed that due to the creation of new release, the first run `bundle exec <COMMAND>` is terribly slow. I was shocked. I had idea that there is something with code initializing or with caching.

I use [Bootsnap](https://github.com/Shopify/bootsnap) for each my Rails application and I've noticed one thing:

```
Note also that Bootsnap will never clean up its own cache: this is left up to you. Depending on your deployment strategy, you may need to periodically purge tmp/cache/bootsnap*. If you notice deploys getting progressively slower, this is almost certainly the cause.
```

Yes, it was the answer! After some years of using Bootsnap on the server it has written caches for 180 Mbs. I have deleted all cache files using `rm tmp/cache/bootsnap*` and It helps me. Be aware, if you are using `Bootsnap` and your application is getting slower on the boost, maybe there is a problem with a large amount of cache.

[dev.to](https://dev.to/kopylov_vlad/why-ruby-app-deploy-is-getting-progressively-slower-46fm)
