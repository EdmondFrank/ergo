<h1><a href="https://ergo.services"><img src=".github/images/logo.svg" alt="Ergo Framework" width="159" height="49"></a></h1>

[![GitHub release](https://img.shields.io/github/release/ergo-services/ergo.svg)](https://github.com/ergo-services/ergo/releases/latest)
[![Go Report Card](https://goreportcard.com/badge/github.com/ergo-services/ergo)](https://goreportcard.com/report/github.com/ergo-services/ergo)
[![GoDoc](https://pkg.go.dev/badge/ergo-services/ergo)](https://pkg.go.dev/github.com/ergo-services/ergo)
[![MIT license](https://img.shields.io/badge/license-MIT-brightgreen.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/github/workflow/status/ergo-services/ergo/TestLinuxWindowsMacOS)](https://github.com/ergo-services/ergo/actions/)

Technologies and design patterns of Erlang/OTP have been proven over the years. Now in Golang.
Up to x5 times faster than original Erlang/OTP in terms of network messaging.
The easiest way to create an OTP-designed application in Golang.

[https://ergo.services](https://ergo.services)

### Purpose ###

The goal of this project is to leverage Erlang/OTP experience with Golang performance. Ergo Framework implements [DIST protocol](https://erlang.org/doc/apps/erts/erl_dist_protocol.html), [ETF data format](https://erlang.org/doc/apps/erts/erl_ext_dist.html) and [OTP design patterns](https://erlang.org/doc/design_principles/des_princ.html) `gen.Server`, `gen.Supervisor`, `gen.Application` which makes you able to create distributed, high performance and reliable microservice solutions having native integration with Erlang infrastructure

### Features ###

![image](https://user-images.githubusercontent.com/118860/113710255-c57d5500-96e3-11eb-9970-20f49008a990.png)

* Support Erlang 24 (including [Alias](https://blog.erlang.org/My-OTP-24-Highlights/#eep-53-process-aliases) and [Remote Spawn](https://blog.erlang.org/OTP-23-Highlights/#distributed-spawn-and-the-new-erpc-module) features)
* Spawn Erlang-like processes
* Register/unregister processes with simple atom
* `gen.Server` behavior support (with atomic state)
* `gen.Supervisor` behavior support with all known [restart strategies](https://erlang.org/doc/design_principles/sup_princ.html#restart-strategy) support
  * One For One
  * One For All
  * Rest For One
  * Simple One For One
* `gen.Application` behavior support with all known [starting types](https://erlang.org/doc/design_principles/applications.html#application-start-types) support
  * Permanent
  * Temporary
  * Transient
* `gen.Stage` behavior support (originated from Elixir's [GenStage](https://hexdocs.pm/gen_stage/GenStage.html)). This is abstraction built on top of `gen.Server` to provide a simple way to create a distributed Producer/Consumer architecture, while automatically managing the concept of backpressure. This implementation is fully compatible with Elixir's GenStage. Example is here [examples/genstage](examples/genstage) or just run `go run ./examples/genstage` to see it in action
* `gen.Saga` behavior support. It implements Saga design pattern - a sequence of transactions that updates each service state and publishes the result (or cancels the transaction or triggers the next transaction step). `gen.Saga` also provides a feature of interim results (can be used as transaction progress or as a part of pipeline processing), time deadline (to limit transaction lifespan), two-phase commit (to make distributed transaction atomic). Here is example [examples/gensaga](examples/gensaga).
* Connect to (accept connection from) any Erlang node within a cluster
* Making sync request `ServerProcess.Call`, async - `ServerProcess.Cast` or `Process.Send` in fashion of `gen_server:call`, `gen_server:cast`, `erlang:send` accordingly
* Monitor processes/nodes
  * local -> local
  * local -> remote
  * remote -> local
* Link processes
  * local <-> local
  * local <-> remote
  * remote <-> local
* RPC callbacks support
* [embedded EPMD](#epmd) (in order to get rid of erlang' dependencies)
* Experimental [observer support](#observer)
* Unmarshalling terms into the struct using `etf.TermIntoStruct`, `etf.TermProplistIntoStruct` or to the string using `etf.TermToString`
* Custom marshaling/unmarshaling via `Marshal` and `Unmarshal` interfaces
* Encryption (TLS 1.3) support (including autogenerating self-signed certificates)
* Tested and confirmed support Windows, Darwin (MacOS), Linux, FreeBSD.

### Requirements ###

* Go 1.15.x and above

### Versioning ###

Golang introduced [v2 rule](https://go.dev/blog/v2-go-modules) a while ago to solve complicated dependency issues. We found this solution very controversial and there is still a lot of discussion around it. So, we decided to keep the old way for the versioning, but have to use the git tag versioning with v1 as a major version (due to "v2 rule" restrictions) . As a starting point for the v2.0.0 we use git tag v1.999.200. Since now, the only "patch version" will be increased for the next releases (e.g. v2.0.1 will be tagged in git as v.1.999.201 and so on, but never be above git tag v1.999 until the moment when Golang developers change the versioning approach)

### Changelog ###

Here are the changes of latest release. For more details see the [ChangeLog](ChangeLog.md)

#### [v2.0.0](https://github.com/ergo-services/ergo/releases/tag/v1.999.200) 2021-10-12 [tag version v1.999.200] ####

* Added support of Erlang/OTP 24 (including [Alias](https://blog.erlang.org/My-OTP-24-Highlights/#eep-53-process-aliases) feature and [Remote Spawn](https://blog.erlang.org/OTP-23-Highlights/#distributed-spawn-and-the-new-erpc-module) introduced in Erlang/OTP 23)
* **Important**: This release includes refined API (without backward compatibility) for a more convenient way to create OTP-designed microservices. Make sure to update your code.
* **Important**: Project repository has been moved to [https://github.com/ergo-services/ergo](https://github.com/ergo-services/ergo). It is still available on the old URL [https://github.com/halturin/ergo](https://github.com/halturin/ergo) and GitHub will redirect all requests to the new one (thanks to GitHub for this feature).
* Introduced new behavior `gen.Saga`. It implements Saga design pattern - a sequence of transactions that updates each service state and publishes the result (or cancels the transaction or triggers the next transaction step). `gen.Saga` also provides a feature of interim results (can be used as transaction progress or as a part of pipeline processing), time deadline (to limit transaction lifespan), two-phase commit (to make distributed transaction atomic). Here is example [examples/gensaga](examples/gensaga).
* Introduced new methods `Process.Direct` and `Process.DirectWithTimeout` to make direct request to the actor (`gen.Server` or inherited object). If an actor has no implementation of `HandleDirect` callback it returns `ErrUnsupportedRequest` as a error.
* Introduced new callback `HandleDirect` in the `gen.Server` interface as a handler for requests made by `Process.Direct` or `Process.DirectWithTimeout`. It should be easy to interact with actors from outside.
* Introduced new types intended to be used to interact with Erlang/Elixir
  * `etf.ListImproper` to support improper lists like `[a|b]` (a cons cell).
  * `etf.String` (an alias for the Golang string) encodes as a binary in order to support Elixir string type (which is `binary()` type)
  * `etf.Charlist` (an alias for the Golang string) encodes as a list of chars `[]rune` in order to support Erlang string type (which is `charlist()` type)
* Introduced new methods `Node.ProvideRemoteSpawn`, `Node.RevokeRemoteSpawn`, `Process.RemoteSpawn`.
* Introduced new interfaces `Marshaler` (method `MarshalETF`) and `Unmarshaler` (method `UnmarshalETF`) for the custom encoding/decoding data.
* Improved performance for the local messaging (up to 3 times for some cases)
* Added example [examples/http](examples/http) to demonsrate how HTTP server can be integrated into the Ergo node.
* Added example [examples/gendemo](examples/gendemo) - how to create a custom behavior (design pattern) on top of the `gen.Server`. Take inspiration from the [gen/stage.go](gen/stage.go) or [gen/saga.go](gen/saga.go) design patterns.
* Added support FreeBSD, OpenBSD, NetBSD, DragonFly.
* Fixed RPC issue #45
* Fixed internal timer issue #48
* Fixed memory leaks #53
* Fixed double panic issue #52
* Fixed Atom Cache race conditioned issue #54
* Fixed ETF encoder issues #64 #66


### Benchmarks ###

Here is simple EndToEnd test demonstrates performance of messaging subsystem

#### Sequential Process.Call using two processes running on a single and two nodes

Hardware: laptop with Intel(R) Core(TM) i5-8265U (4 cores. 8 with HT)

```
❯❯❯❯ go test -bench=NodeSequential -run=XXX -benchtime=10s
goos: linux
goarch: amd64
pkg: github.com/ergo-services/ergo
BenchmarkNodeSequential/number-8 	  256108	     48578 ns/op
BenchmarkNodeSequential/string-8 	  266906	     51531 ns/op
BenchmarkNodeSequential/tuple_(PID)-8         	  233700	     58192 ns/op
BenchmarkNodeSequential/binary_1MB-8          	    5617	   2092495 ns/op
BenchmarkNodeSequentialSingleNode/number-8         	 2527580	      4857 ns/op
BenchmarkNodeSequentialSingleNode/string-8         	 2519410	      4760 ns/op
BenchmarkNodeSequentialSingleNode/tuple_(PID)-8    	 2524701	      4757 ns/op
BenchmarkNodeSequentialSingleNode/binary_1MB-8     	 2521370	      4758 ns/op
PASS
ok  	github.com/ergo-services/ergo	120.720s
```

it means Ergo Framework provides around **25.000 sync requests per second** via localhost for simple data and around 4Gbit/sec for 1MB messages

#### Parallel Process.Call using 120 pairs of processes running on a single and two nodes

Hardware: workstation with AMD Ryzen Threadripper 3970X (64) @ 3.700GHz

```
❯❯❯❯ go test -bench=NodeParallel -run=XXX -benchtime=10s
goos: linux
goarch: amd64
pkg: github.com/ergo-services/ergo/tests
cpu: AMD Ryzen Threadripper 3970X 32-Core Processor
BenchmarkNodeParallel-64                 4922430              2440 ns/op
BenchmarkNodeParallelSingleNode-64      16293586               810.0 ns/op
PASS
ok      github.com/ergo-services/ergo/tests  29.596s
```

these numbers show almost **500.000 sync requests per second** for the network messaging via localhost and **1.600.000 sync requests per second** for the local messaging (within a node).

#### Ergo Framework vs original Erlang/OTP

Hardware: laptop with Intel(R) Core(TM) i5-8265U (4 cores. 8 with HT)

![benchmarks](https://raw.githubusercontent.com/halturin/ergobenchmarks/master/ergobenchmark.png)

sources of these benchmarks are [here](https://github.com/halturin/ergobenchmarks)


### EPMD ###

*Ergo Framework* has embedded EPMD implementation in order to run your node without external epmd process needs. By default, it works as a client with erlang' epmd daemon or others ergo's nodes either.

The one thing that makes embedded EPMD different is the behavior of handling connection hangs - if ergo' node is running as an EPMD client and lost connection, it tries either to run its own embedded EPMD service or to restore the lost connection.

### Observer ###

It's a standard Erlang tool. Observer is a graphical tool for observing the characteristics of Erlang systems. The tool Observer displays system information, application supervisor trees, process information.

Here you can see this feature in action using one of the [examples](examples/):

![observer demo](.github/images/observer.gif)

### Examples ###

Code below is a simple implementation of gen.Server pattern [examples/simple](examples/simple)

```golang
package main

import (
	"fmt"
	"time"

	"github.com/ergo-services/ergo"
	"github.com/ergo-services/ergo/etf"
	"github.com/ergo-services/ergo/gen"
	"github.com/ergo-services/ergo/node"
)

// simple implementation of Server
type simple struct {
	gen.Server
}

func (s *simple) HandleInfo(process *gen.ServerProcess, message etf.Term) gen.ServerStatus {
	value := message.(int)
	fmt.Printf("HandleInfo: %#v \n", message)
	if value > 104 {
		return gen.ServerStatusStop
	}
	// sending message with delay
	process.SendAfter(process.Self(), value+1, time.Duration(1*time.Second))
	return gen.ServerStatusOK
}

func main() {
	// create a new node
	node, _ := ergo.StartNode("node@localhost", "cookies", node.Options{})

	// spawn a new process of gen.Server
	process, _ := node.Spawn("gs1", gen.ProcessOptions{}, &simple{})

	// send a message to itself
	process.Send(process.Self(), 100)

	// wait for the process termination.
	process.Wait()
	fmt.Println("exited")
	node.Stop()
}

```

here is output of this code

```shell
$ go run ./examples/simple
HandleInfo: 100
HandleInfo: 101
HandleInfo: 102
HandleInfo: 103
HandleInfo: 104
HandleInfo: 105
exited
```

See `examples/` for more details

* [gen.Application](examples/application)
* [gen.Supervisor](examples/supervisor)
* [gen.Server](examples/genserver)
* [gen.Stage](examples/genstage)
* [gen.Saga](examples/gensaga)
* [gen.Demo](examples/gendemo)
* [Node with TLS](examples/nodetls)
* [Node with HTTP server](examples/http)

### Elixir Phoenix Users ###

Users of the Elixir Phoenix framework might encounter timeouts when trying to connect a Phoenix node
to an ergo node. The reason is that, in addition to global_name_server and net_kernel,
Phoenix attempts to broadcast messages to the [pg2 PubSub handler](https://hexdocs.pm/phoenix/1.1.0/Phoenix.PubSub.PG2.html)

To work with Phoenix nodes, you must create and register a dedicated pg2 GenServer, and
spawn it inside your node. The spawning process must have "pg2" as a process name:

```golang
type Pg2GenServer struct {
    gen.Server
}

func main() {
    // ...
    pg2 := &Pg2GenServer{}
    node1, _ := ergo.StartNode("node1@localhost", "cookies", node.Options{})
    process, _ := node1.Spawn("pg2", gen.ProcessOptions{}, pg2, nil)
    // ...
}
```

### Development and debugging ###

There are options already defined that you might want to use

* `-ergo.trace` - enable extended debug info
* `-ergo.norecover` - disable panic catching

To enable Golang profiler just add `--tags debug` in your `go run` or `go build` like this:

```
go run --tags debug ./examples/genserver/demoGenServer.go
```

Now golang' profiler is available at `http://localhost:9009/debug/pprof`

To check test coverage:

```
go test -coverprofile=cover.out ./...
go tool cover -html=cover.out -o coverage.html
```

To run tests with cleaned test cache:

```
go vet
go clean -testcache
go test -v ./...
```

To run benchmarks:

```
go test -bench=Node -run=X -benchmem
```

### Companies are using Ergo Framework ###

[![Kaspersky](.github/images/kaspersky.png)](https://kaspersky.com)
[![RingCentral](.github/images/ringcentral.png)](https://www.ringcentral.com)
[![LilithGames](.github/images/lilithgames.png)](https://lilithgames.com)

is your company using Ergo? add your company logo/name here

### Commercial support

please, visit https://ergo.services for more information
