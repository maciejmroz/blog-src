+++
date = "2017-02-05T13:17:09+01:00"
title = "Implementing password storage with Golang"
slug = "implementing-password-storage-with-golang"
categories = ["Programming", "Server Side", "Security"]
tags = ["Golang"]

+++

The common wisdom says: "Do not implement password storage on your own" ... Considering amount of things that can be done wrong, perhaps relying on one of plenty of external providers is the right thing to do :) If you still want to do this, read [OWASP Password Storage Cheat Sheet](https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet) as a first step. After that, read [OWASP Authentication Cheat Sheet](https://www.owasp.org/index.php/Authentication_Cheat_Sheet) - password storage and authentication are close layers and most likely you'll be working on both. 

What modern password storage attempts to achieve is to make life harder for an attacker when password database **is already compromised**. In the era of GPU/FPGA password cracking MD5 will not suffice. This may sound trivial, but apparently [it's not](https://twitter.com/maciejmroz/status/809396388597563392).

Jokes aside, at the very minimum use:

* computationally expensive password hashing function, preferably one with tunable cost
* dedicataded random salt for every credential you store

Password storage for every credential should contain:

* cost - so that if you decide that it is too low compared to what computing power potential attacker has at his/her disposal (you can assume it's proportional to your business value). 
* random salt - I'll restate it: unique for every user, stored together with hashed password. Salt **is not** an encryption measure, if your db is compromised, so is the app, and so is the salt. What it really protects from is "rainbow table" kind of attack. With unique salt for every credential you also make it impossible to hash en entire dictionary and fish for something that matches.
* hashed password (obviously salted :) )

Good news and bad news coming. Good one: if you use modern platform/framework, it is likely that there exist libraries that take care of the above. In case of Go, you have [golang.org/x/crypto/bcrypt](https://godoc.org/golang.org/x/crypto/bcrypt) package for that. It is trivial to use, and contrary to some StackOverflow answers there's absolutely no need to add salt in your own code - the library is already doing it for you.

Now time for the bad news, or to be more specific, bad news and a little example of why Go is an awesome language. The problem is, by using computationally expensive hashing function, you expose your website to an extremely easy denial of service attack. All that needs to be done to put website on its knees is to flood it with login requests - in typical scenario it will consume all your CPU resources and bring all your servers to a halt. And this is vulnerability we introduced ourselves: in the world of thousand requests per second we intentionally made part of the application very, very slow.

How to deal with it? There are many ways, and probably more than one should be used. Below I'll show how to limit resources used by delegating expensive processing to a configurable pool of goroutines, and show how to introduce backpressure in this design.

At the highest level, in the service initialization code create channels used to communicate with goroutines and run requested number of them:

{{< highlight go >}}

func NewAccountService(db *mgo.Database, BCryptCost int, BcryptQueueDepth int, BcryptConcurrency int) AccountService {

    (...)

    as := &accountService{
	    BcryptCost:        BCryptCost,
	    fromPasswordQueue: make(chan generateFromPasswordRequest, BcryptQueueDepth),
	    compareQueue:      make(chan compareHashAndPasswordRequest, BcryptQueueDepth),
    }

    for i := 0; i < BcryptConcurrency; i++ {
	    go bcryptRequestHandler(as.fromPasswordQueue, as.compareQueue)
    }

    return as
}

{{< /highlight>}}

It is important that buffered channels are used above. It allows certain number of requests to be enqueued, and allows us to respond properly when channel is full. Simply blocking on full channel is not desireable because we run a goroutine per request and in case of DoS attack we'll limit the CPU but will continue to leak memory by spawning tons of goroutines (or hit hard limit of the web server and become inaccessible). I'll show how to deal with that later. The request/response types are really simple, as the execution goroutine is very thin wrapper over bcrypt:

{{< highlight go >}}

type generateFromPasswordResult struct {
	hashedPassword []byte
	err            error
}

type generateFromPasswordRequest struct {
	password   []byte
	BCryptCost int

	result chan generateFromPasswordResult
}

type compareHashAndPasswordRequest struct {
	password       []byte
	hashedPassword []byte

	result chan error
}

func bcryptRequestHandler(fromPasswordQueue chan generateFromPasswordRequest, compareQueue chan compareHashAndPasswordRequest) {
	for {
		select {
		case rq := <-fromPasswordQueue:
			hashedPassword, err := bcrypt.GenerateFromPassword(rq.password, rq.BCryptCost)
			rq.result <- generateFromPasswordResult{hashedPassword, err}
		case rq := <-compareQueue:
			rq.result <- bcrypt.CompareHashAndPassword(rq.hashedPassword, rq.password)
		}
	}
}

{{< /highlight>}}

Pretty neat thing about Go: you send channel over which response is sent together with the request. While this is a common pattern in Go, people unfamiliar with Go concurrency and CSP model assume that communication topology must be static. In fact, ephemeral constructs like this one are quite normal in Go. You can also see it as a similarity to sending return address along with Akka actor message. After all, these models are computationally equivalent, just with different semantics. By the way, if you look for other platform that easily allows you to pin some processing to a group of threads, yes, JVM is great at this stuff, and no, node.js can't do it :)

Ok, let's introduce backpressure:

{{< highlight go >}}

func (s *accountService) newFromPasswordRequest(password string) (chan generateFromPasswordResult, error) {
	if len(s.fromPasswordQueue) >= cap(s.fromPasswordQueue) {
		return nil, ErrOverload
	}
	rq := generateFromPasswordRequest{
		password:   []byte(password),
		BCryptCost: s.BcryptCost,
		result:     make(chan generateFromPasswordResult, 0),
	}
	s.fromPasswordQueue <- rq
	return rq.result, nil
}

func (s *accountService) newCompareRequest(hashedPassword string, password string) (chan error, error) {
	if len(s.compareQueue) >= cap(s.compareQueue) {
		return nil, ErrOverload
	}
	rq := compareHashAndPasswordRequest{
		hashedPassword: []byte(hashedPassword),
		password:       []byte(password),
		result:         make(chan error, 0),
	}
	s.compareQueue <- rq
	return rq.result, nil
}

{{< /highlight>}}

The functions above return channel or error (unfortunately there's no equivalent of Scala's Either[A,B] type in Go). The magic is in comparison *len(channel) >= cap(channel)*: if we ever reach channel capacity, just return error indicating overload and let upstream code decide what to do about it (the approach will likely be different in a user facing website and different in a REST service consumed by a machine but in any case should be some sort of "try again later" signal). From the perspective of code that makes use of our wrappers, things are pretty simple:

{{< highlight go >}}

//hashing password
(...)
resChannel, err := s.newFromPasswordRequest(password)
if err != nil {
    return nil, err
}
res := <-resChannel
hashedPassword, err := res.hashedPassword, res.err
if err != nil {
    return nil, err
}
(...)

//checking password against hashed value
(...)
resChannel, err := s.newCompareRequest(account.HashedPassword, password)
if err != nil {
    return nil, err
}
err = <-resChannel
if err != nil {
    return nil, err
}
(...)

{{< /highlight>}}

Obviously, a proper password storage and authentication is going to be much, much more complicated :) Also, the problem of protecting yourself from DoS or DDoS is much more complex, and a lot of measures need to be taken, not only at the application layer. Still, I consider this pattern of putting a controllable upper bound on resources consumed by certain module in the application **without breaking it into microservices** very useful, especially when combined with proper profiling and monitoring.