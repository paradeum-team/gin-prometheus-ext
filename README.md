# gin-prometheus-ext

在[go-gin-prometheus](https://github.com/zsais/go-gin-prometheus) 的基础上，增加了几个metrics ，兼容api的分位数和耗时统计



# go-gin-prometheus
[![](https://godoc.org/github.com/zsais/go-gin-prometheus?status.svg)](https://godoc.org/github.com/zsais/go-gin-prometheus) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Gin Web Framework Prometheus metrics exporter

## Installation

`$ go get github.com/paradeum-team/gin-prometheus-ext`

## Usage

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/paradeum-team/gin-prometheus-ext"
	"strings"
)

func main() {
	r := gin.New()

	p := ginprometheusext.NewPrometheus("gin")
	p.Use(r)

	r.GET("/", func(c *gin.Context) {
		c.JSON(200, "Hello world!")
	})

	r.Run(":8090")
}
```


## Preserving a low cardinality for the request counter

The request counter (`requests_total`) has a `url` label which,
although desirable, can become problematic in cases where your
application uses templated routes expecting a great number of
variations, as Prometheus explicitly recommends against metrics having
high cardinality dimensions:

https://prometheus.io/docs/practices/naming/#labels

If you have for instance a `/customer/:name` templated route and you
don't want to generate a time series for every possible customer name,
you could supply this mapping function to the middleware:

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/paradeum-team/gin-prometheus-ext"
	"strings"
)

func main() {
	r := gin.New()

	p := ginprometheusext.NewPrometheus("gin")

	p.ReqCntURLLabelMappingFn = func(c *gin.Context) string {
		url := c.Request.URL.Path
		for _, p := range c.Params {
			if p.Key == "name" {
				url = strings.Replace(url, p.Value, ":name", 1)
				break
			}
		}
		return url
	}

	p.Use(r)

	r.GET("/", func(c *gin.Context) {
		c.JSON(200, "Hello world!")
	})

	r.Run(":8090")
}
```

which would map `/customer/alice` and `/customer/bob` to their
template `/customer/:name`, and thus preserve a low cardinality for
our metrics.
