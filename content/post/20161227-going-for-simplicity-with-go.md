+++
title = "Going for simplicity with Go"
date = "2016-12-27T17:03:44+01:00"
slug = "going-for-simplicity-with-go"
categories = ["Programming", "Server Side"]
tags = ["Golang", "HTTPS", "HSTS", "Let's Encrypt", "Echo"]
+++

If you are backend engineer, you probably treat nonfunctional requirements like security, availability, scalability etc. just as if they were laws of physics. And that's good when working on mature, mission critical systems. But what if you don't need all that and want to simply focus on getting something simple but professionally looking out there? Something like MVP/early stage product?

Do you really need microservices, containers, service discovery, load balancers, caches, multiple databases, all clustered and replicated accross multiple availability zones? Hint: you are not making any money yet and there's big chance you will not get to the point where you have product/market fit.

So, let's take a step back and optimize for something else entirely: ease and speed of development. There are many possible ways to approach a problem framed this way. I'll show how to do that using Go language and [Labstack Echo](https://echo.labstack.com/) framework.

I initially thought of putting the code on GitHub, but essentially it's a compilation of few samples from Echo docs, so I'll just paste it here:

{{< highlight go >}}

package main

import (
	"flag"

	"golang.org/x/crypto/acme/autocert"

	"github.com/labstack/echo"
	"github.com/labstack/echo/middleware"
)

func redirectHTTP() {
	e := echo.New()
	e.Use(middleware.HTTPSRedirect())
	go func() { e.Logger.Fatal(e.Start(":80")) }()
}

func main() {

	devFlagPtr := flag.Bool("dev", false, "Run on dev port, no TLS")
	flag.Parse()

	e := echo.New()
	e.Use(middleware.Gzip())

	e.Static("/css", "static/css")
	e.Static("/fonts", "static/fonts")
	e.Static("/js", "static/js")
	e.File("/", "static/index.html")

	if *devFlagPtr {
		e.Logger.Fatal(e.Start(":1323"))
	} else {
		redirectHTTP()
		e.Use(middleware.SecureWithConfig(middleware.SecureConfig{
			HSTSMaxAge: 63072000,
		}))
		e.AutoTLSManager.Cache = autocert.DirCache(".")
		e.AutoTLSManager.HostPolicy = autocert.HostWhitelist("cs.maciejmroz.com")
		e.Logger.Fatal(e.StartAutoTLS(":443"))
	}
}

{{< /highlight>}}

What you see above is a concurrent http server serving content over HTTPS with certificate obtained automatically from [Let's Encrypt](https://letsencrypt.org/), also bound to port 80 redirecting everything there to 443 (where [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) headers are served), all with support for TLS 1.2 and HTTP/2. When deployed to Digital Ocean droplet, it is reachable over IPv6 and gets A+ grade from [SSL Labs](https://www.ssllabs.com/).

There are more considerations if you want to put [Go-based website in production](https://blog.gopheracademy.com/advent-2016/exposing-go-on-the-internet/) but this is nice starting point for something more featured.

## Where are we?

Pretty much no moving parts except main executable:

 * **No load balancer** in front. Go performance is high enough you will not need horizontal scaling for a very long time. And you don't need HA yet.
 * **No web/application server**. Another piece of software you don't need. Go supports Websockets, HTTP/2 and TLS 1.2 out of the box. Echo makes it trivial to acquire certificate from Let's Encrypt. 
 * **No runtime**. Go is a compiled language and everything is statically linked. You don't need to worry about version of PHP/Python/Ruby/node installed on your server, and whether the libraries are in correct versions. For Go, it's all build time affair, what you deploy is ELF binary + static files (in fact, there are solutions to bundle these togother, too). There is really no need to bother with packaking the app inside a Docker (or rkt, or whatever) container. Just run single binary directly from systemd, with config file like this:

{{< highlight ini >}}
[Unit]
Description=Go web app
Documentation=https://cs.maciejmroz.com/
After=network.target

[Service]
User=app
WorkingDirectory=/opt/app/
LimitNOFILE=4096
ExecStart=/opt/app/app-linux-amd64
Restart=on-failure
StartLimitInterval=60

[Install]
WantedBy=multi-user.target
{{< /highlight>}}

 * You don't even need Linux machine/Vagrant VM to build the binary thanks to excellent cross compilation support in Go. It just works, here's the Makefile I use on Mac OS X so I don't need to type too much: 
 
{{< highlight makefile >}}
all:
	GOOS=linux GOARCH=amd64 go build -o app-linux-amd64
{{< /highlight>}}

 * A tip for the less savvy on modern Linux, in order to bind non-root process to port <1024, you need something like this (as root or via sudo, of course :) ):

{{< highlight bash >}}
setcap 'cap_net_bind_service=+ep' /opt/app/app-linux-amd64
{{< /highlight>}}

## What next?

* Application level caching is easy - at the simplest you can get any concurrent cache out there and just do it in-process. No HA, remember? Biggest instance on Digital Ocean has 64 GB RAM, on Amazon you can get many, many times more. Eventually you will move towards something like [groupcache](https://github.com/golang/groupcache) also solving "thundering herd" problem properly.
* How about async jobs? RabbitMQ to the rescue, right? Wrong. Less moving parts, remember? In Go, when you want to launch an async job you launch a goroutine, or communicate with one that already exists via channel. Point is, you use concurrency features of the language. You don't need to get truly distributed yet - you get up to 20 cores on DO, more on Amazon. When you run compiled language with good concurrency support, that's a lot.
* At the database level I'd actually backtrack a bit a suggest you run MongoDB yourself. It's not much work to set it up and productivity on the app level is worth it. It sounds tempting to go with 100% managed solution, but without traffic and paying customers in my opinion it's not really worth it. I'm not religious on this point - i.e. if you are fluent with DynamoDB and do your hosting on Amazon, or perhaps use some equivalent db service on the GCE, by all means go for it.


## Ending words

Nobody builds with LAMP stack any more - there are plenty of other ways. MEAN (MongoDB, Express, Angular, Node) seems to be chosen by a lot of people. If you are strong with JavaScript/TypeScript it is not really a bad choice. But it's not the only one out there. I hope that this post will get someone at least a bit curious about the alternatives :)
