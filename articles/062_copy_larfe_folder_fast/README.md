# How to copy a large folder fast

Having and supporting a large and old project can be very funny. Recently I've noticed one line in Capistrana deploy logs:

```
cp -pr node_modules tmp/node_modules
240.791s
```

The old project has grown and now the stage of copying the folder `node_modules` lasts 4 minutes. I had an idea to speed it up. First idea was to create a symlink instead of copying the folder. I've changed the line.

```
ls -s node_modules tmp/node_modules
```

It can be a good solution, but in our project we are using an old version of yarn and it can't [handle it](https://github.com/yarnpkg/yarn/issues/8441).

```
yarn install
[1/4] 🔍  Resolving packages...
[2/4] 🚚  Fetching packages...
error An unexpected error occurred: "ELOOP: too many symbolic links encountered
```

Next idea was to compress all node modules and decompress the folder. It can help, but it depends on the filesystem. I'be changed the script again.

```
tar -czf modules.tar.gz node_modules/.
tar -xzf modules.tar.gz tmp/
```

Fortunately, it helps me. The stage passes by 30 seconds, it's better than over 200 seconds. Remember about it that some folders on your server can get bigger and it can make your deployment slower.
