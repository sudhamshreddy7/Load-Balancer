# GoLang Load Balancer

A load balancer acts as a reverse proxy, distributing workloads efficiently across multiple servers to optimize computing performance. It routes incoming requests to one of several backend servers, aiming to maximize speed and resource utilization.

## Contents
- [Load Balancing Algorithms](#load-balancing-algorithms)
- [Architecture Overview](#architecture-overview)
- [Usage Examples](#usage-examples)

---

## Load Balancing Algorithms

The load balancer currently supports several algorithms to distribute traffic:
- **Round Robin**: Distributes requests sequentially across servers.
- **Weighted Round Robin**: Assigns weights to servers, sending more requests to higher-weighted servers.
- **Random**: Selects servers randomly to distribute requests.
- **Least Connections**: Routes requests to the server with the fewest active connections.

## Architecture Overview

The design is intentionally simplified to focus on key concepts:

![Architecture Schema](https://i.ibb.co/tPJT5WN/Screenshot-2020-01-10-at-14-12-55.png)

## Usage Examples

### Round Robin
The Round Robin algorithm distributes requests evenly among the servers in a circular order.

```go
func main() {
	shortRespUrl, err := url.Parse("http://127.0.0.1:8081")
	if err != nil {
		log.Fatal(err)
	}
	p1 := proxy.NewProxy(shortRespUrl)

	longRespUrl, err := url.Parse("http://127.0.0.1:8082")
	if err != nil {
		log.Fatal(err)
	}
	p2 := proxy.NewProxy(longRespUrl)

	lb := loadbalancer.NewLoadBalancer(iterator.NewRoundRobin(p1, p2))
	log.Printf("Load balancer running on port :8080")
	go func() {
		log.Fatal(http.ListenAndServe(":8080", lb))
	}()

	for i := 0; i < 5; i++ {
		func() {
			r, _ := http.Get("http://127.0.0.1:8080")
			b, _ := ioutil.ReadAll(r.Body)
			log.Printf("Received response %d: %s", i, string(b))
		}()
	}
}
```

**Output:**
```
2019/12/04 11:59:23 Load balancer running on port :8080
2019/12/04 11:59:24 Received response 0: --- short resp ---
2019/12/04 11:59:24 Received response 1: --------- long resp ---------
2019/12/04 11:59:24 Received response 2: --- short resp ---
2019/12/04 11:59:24 Received response 3: --------- long resp ---------
2019/12/04 11:59:24 Received response 4: --- short resp ---
```

### Weighted Round Robin
This approach assigns weights to each server, determining the frequency with which they handle requests.

```go
func main() {
	// ...

	lb := loadbalancer.NewLoadBalancer(iterator.NewWeightedRoundRobin(map[*proxy.Proxy]int32{
		p1: 3, // p1 will handle 3 times more requests than p2
		p2: 1,
	}))
	
	// ...
}
```

**Output:**
```
2020/01/14 11:44:30 Load balancer running on port :8080
2020/01/14 11:44:30 Received response 0: --- short resp ---
2020/01/14 11:44:30 Received response 1: --- short resp ---
2020/01/14 11:44:30 Received response 2: --- short resp ---
2020/01/14 11:44:30 Received response 3: --------- long resp ---------
2020/01/14 11:44:30 Received response 4: --- short resp ---
```

### Random
The Random algorithm assigns requests randomly to servers. The seed function can be customized to control the randomness, which is useful for debugging.

```go
func main() {
	// ...

	seed := func() {
		rand.Seed(time.Now().UTC().UnixNano())
	}
	lb := loadbalancer.NewLoadBalancer(iterator.NewRandom(seed, p1, p2))

	// ...
}
```

### Least Connections
This algorithm selects the server with the fewest active connections, helping balance the load dynamically.

```go
	// ...

	lb := loadbalancer.NewLoadBalancer(iterator.NewLeastConnections(p1, p2))

	// ...
```

These examples demonstrate how to create and use various load balancing strategies with GoLang, ensuring efficient and fair distribution of network traffic.