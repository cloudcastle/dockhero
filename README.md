[DockHero](http://addons.heroku.com/dockhero) hosts your Docker stack in AWS cloud.

If you think of Heroku add-ons in general as of boxes with some useful mechanisms inside \(like databases, log analyzers, messengers etc.\), then Dockhero is an empty box where you can put your own microservice described by [docker-compose.yml](https://docs.docker.com/compose/compose-file/).

![](https://s3.amazonaws.com/heroku-devcenter-files/article-images/2812-imported-1491376744-heroku_dockhero_2_padding.png)

When you add Dockhero add-on to your Heroku application, a new Docker cluster \(currently consisting of a single Swarm master\) is provisioned, and it's address is exposed to your Heroku app via DOCKHERO\_HOST environment variable.

## Provisioning the add-on

DockHero can be attached to a Heroku application via the CLI:

> callout
> A list of all plans available can be found [here](http://addons.heroku.com/dockhero).

```term
$ heroku addons:create dockhero
-----> Adding dockhero to sharp-mountain-4005... done, v18 (free)
```

The actual provisioning may take up to 3 minutes. You can track the provisioning progress by logging into add-on dashboard:

```term
$ heroku addons:open dockhero
```

or using CLI plugin:

```term
$ heroku dh:wait  # requires CLI plugin installed - see below
```

Once provisioned, you can find the address of your Docker Swarm among Heroku env variables:.

```term
$ heroku config:get DOCKHERO_HOST
orange-apple-1984.dockhero.io
```

## Installing CLI plugin

In order to access the newly provisioned Swarm cluster, install Dockhero CLI plugin:

```term
  heroku plugins:install dockhero
```

Now you can run docker / docker-compose against that cluster using the shortcuts below:

```term
  heroku dh:docker
  heroku dh:compose
```

Find other available commands in the [docs](https://github.com/cloudcastle/dockhero-cli)


## Preparing a stackfile

Use the following command to generate a sample stackfile:

```
$ heroku dh:generate helloworld
Writing example stack into dockhero-compose.yml...
```

This will create  **dockhero-compose.yml** in the root of your repo.
It's recommended that you put this file under version control.
See [docker-compose.yml reference](https://docs.docker.com/compose/compose-file/) for the file syntax.
There are also [stackfile examples](https://github.com/cloudcastle/dockhero-docs/tree/master/examples) available for your reference.

In our example below, we'll be building a pluggable resource which returns message of the day via HTTP.
Here's a [Docker image](https://hub.docker.com/r/dockhero/motd-http/) which does that.

> callout
> You should already have Docker client set up locally.
> Please follow the [installation instructions for your platform](https://docs.docker.com/installation/)

First, replace `dockhero-compose.yml` content with the following:

```yaml
web:
  image: dockhero/motd-http
  ports:
    - "80:8000"
```

Then run your stack:

```term
$ heroku dh:compose up -d
Creating network "dockhero_default" with driver "bridge"
Creating dockhero_web_1

$ heroku dh:compose ps
Name                  Command             State             Ports            
---------------------------------------------------------------------------------
dockhero_web_1   /bin/sh -c http-server /app   Up      54.174.36.199:80->8080/tcp
```

Now you should be able to see a "Message Of The Day" by opening the app in your browser

```term
$ heroku dh:open
Opening http://orange-apple-1984.dockhero.io/...
```

Find other useful commands in [docker-compose cli reference](https://docs.docker.com/compose/reference/)

## Using from Heroku app

In an application running on Heroku, you can fetch the host name of your Docker service from DOCKHERO_HOST environment variable. You can read the MOTD message from our example stack using the following Ruby code:

```ruby
#!/usr/bin/env ruby
require 'open-uri'
response = open("http://#{DOCKHERO_HOST}/") { |f| f.read }
puts "DOCKHERO says:\n  #{response}"
```

## Using SSL Endpoint

Each Dockhero cluster comes with two CloudFlare endpoints configured: one in *flexible ssl* mode and another in *full ssl* mode. They have valid SSL certificate installed, so that you don't need to purchase one yourself. Both endpoints are exposed to your Heroku environment:

```term
$ heroku config:get DOCKHERO_FLEXIBLE_SSL_URL
$ heroku config:get DOCKHERO_FULL_SSL_URL
```

SSL is terminated at the CloudFlare edge server, then the request is sent to Dockhero cluster via http:// or https:// protocol depending on SSL mode (flexible or full).

With Flexible SSL, you don't need to implement SSL in your stack at all.

![Flexible SSL](https://s3.amazonaws.com/heroku-devcenter-files/article-images/2812-imported-1491376746-cfssl_flexible.png)

With Full SSL, your stack still needs to talk SSL, but you can use a self-signed certificate. No worries, the users will see a valid CloudFlare's certificate - find more about CloudFlare SSL in [this article](https://support.cloudflare.com/hc/en-us/articles/200170416-What-do-the-SSL-options-mean-)

![Full SSL](https://s3.amazonaws.com/heroku-devcenter-files/article-images/2812-imported-1491376747-cfssl_full.png)


## Logging

DockHero activity can be identified within the Heroku log-stream by `dockhero` prefix:

```term
$ heroku logs -p dockhero
```

You can also attach to Docker's logs stream directly:

```
$ heroku dh:compose logs --follow
```

## Troubleshooting known issues

You can run your stack in foreground mode to simplify debugging.
Remember to pull updated images and pass `--build` and `--force-recreate` options
in order for changes to apply:

```term
$ heroku dh:compose pull
Pulling web (dockhero/dockhero-docs:hello)...
modern-fog-4352: Pulling dockhero/dockhero-docs:hello... : downloaded

$ heroku dh:compose up
Creating dockhero_web_1
Attaching to dockhero_web_1
web_1  | Starting up http-server, serving /app
web_1  | Available on:
web_1  |   http://127.0.0.1:8080
web_1  |   http://172.17.0.4:8080
web_1  | Hit CTRL-C to stop the server
```

> callout
> After stopping your stack (especially with Ctrl-C), remember to wait until
> it actually goes down and reports this in the console. It usually takes 5-20 seconds.



## Migrating between plans

Currently only the Test plan is supported

## Removing the add-on

DockHero can be removed via the CLI.

> warning
> This will destroy all associated data and cannot be undone!

```term
$ heroku addons:destroy dockhero
-----> Removing dockhero from sharp-mountain-4005... done, v20 (free)
```


## Support

All DockHero support and runtime issues should be submitted via one of the [Heroku Support channels](support-channels). For other questions or suggestions, please email us at [dockhero@castle.co](mailto: dockhero@castle.co).

You can improve the current docs or post your own stackfile examples by forking [our repo](https://github.com/dockhero/generators/) and sending a pull request.
