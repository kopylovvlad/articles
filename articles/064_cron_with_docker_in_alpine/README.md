# How to run CRON with Docker in alpine

In my job, we are using alpine docker images for each application. In order to implement new feature we have installed CRON in a docker image. We have noticed that CRON does not work. In logs we saw the error:

```
crond: can't set groups: Operation not permitted
```

It was unusual bug. I have found that in alpine CRON must be running as the [root privilege](https://github.com/inter169/systs/blob/master/alpine/crond/README.md#run-cron-as-non-root-on-alpine-linux). It's harmful for security concerns. In order to run CRON as the normal user privilege there are two solutions:

1. Using [geekidea/alpine-cron](https://hub.docker.com/r/geekidea/alpine-cron) docker images
2. Choose another linux distributive


The choice is yours...

[dev.to](https://dev.to/kopylov_vlad/how-to-run-cron-with-docker-in-alpine-3b05)
