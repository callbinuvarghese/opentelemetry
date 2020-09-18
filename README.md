# opentelephony

https://epsagon.com/tools/introduction-to-opentelemetry-overview/
https://epsagon.com/observability/opentelemetry-best-practices-overview-part-2-2/

https://developer.ibm.com/components/appsody/tutorials/distributed-tracing-for-microservices-1/#2-create-the-node-js-application


➜ brew tap appsody/appsody
➜ brew install appsody
➜ appsody list

➜ stack_hub_url=https://github.com/kabanero-io/kabanero-stack-hub/releases/latest/download/kabanero-stack-hub-index.yaml\n\nappsody repo add kabanero ${stack_hub_url}
➜ appsody list kabanero

Network isolation
Create a separate docker network. You associate containers to use the network with --network parameter in docker run.
Containers can communicate within networks, but not across networks.
A container with multiple network attachments can connect with all of the container in all of those networks associated.
Bridge, Host and none networks are created by default by docker.
Bridge network is the default network.

➜ docker network create opentrace_network
➜   docker network ls
NETWORK ID          NAME                        DRIVER              SCOPE
62acea84e0f2        bridge                      bridge              local
25035d86efb4        host                        host                local
962bf246547a        none                        null                local
96ba2066c33b        opentrace_network           bridge              local
➜   docker network inspect opentrace_network
[
    {
        "Name": "opentrace_network",
        "Id": "96ba2066c33bae68683525ab01c84759f520ed7c49fe0ca4392093131c469199",
        "Created": "2020-09-17T17:59:57.9696572Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.22.0.0/16",
                    "Gateway": "172.22.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "012bf0fe009ca67a97178a62f52441b160ce7311cd3ab0f767e132fafe50c7bb": {
                "Name": "springboot-tracing",
                "EndpointID": "15b33cde963338179c68ab286fa2d749336c6a18c2c2b90903cded993e15d913",
                "MacAddress": "02:42:ac:16:00:04",
                "IPv4Address": "172.22.0.4/16",
                "IPv6Address": ""
            },
            "1f2c37bdd1e5483a518ca9070da8157f5d2383a3ad215645e8553bbbfa085d66": {
                "Name": "jaeger-collector",
                "EndpointID": "588cefcb285c1b932d8a7d3c72bb152544680f793a88530ffcbcb6d1559e19e7",
                "MacAddress": "02:42:ac:16:00:02",
                "IPv4Address": "172.22.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]



https://www.jaegertracing.io/docs/1.19/getting-started/
Jaeger vs Zipkin
Jaeger - CNCF created
Zipkin - Twitter



➜  docker run --name jaeger-collector \  
--detach \
  --rm \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 9411:9411 \
  --network opentrace_network \
 jaegertracing/all-in-one:latest

Port	Protocol	Component	Function
5775	UDP	agent	accept zipkin.thrift over compact thrift protocol (deprecated, used by legacy clients only)
6831	UDP	agent	accept jaeger.thrift over compact thrift protocol
6832	UDP	agent	accept jaeger.thrift over binary thrift protocol
5778	HTTP	agent	serve configs
16686	HTTP	query	serve frontend
14268	HTTP	collector	accept jaeger.thrift directly from clients
14250	HTTP	collector	accept model.proto
9411	HTTP	collector	Zipkin compatible endpoint (optional)

➜ docker ps
1f2c37bdd1e5        jaegertracing/all-in-one:latest               "/go/bin/all-in-one-…"   24 hours ago        Up 24 hours         0.0.0.0:5775->5775/udp, 0.0.0.0:5778->5778/tcp, 0.0.0.0:9411->9411/tcp, 0.0.0.0:14268->14268/tcp, 0.0.0.0:6831-6832->6831-6832/udp, 0.0.0.0:16686->16686/tcp, 142
50/tcp   jaeger-collector
c57

docker logs jaeger-collector
{"level":"info","ts":1600365618.2730315,"caller":"app/server.go:163","msg":"Query server started","port":16686,"addr":":16686"}
{"level":"info","ts":1600365618.27311,"caller":"healthcheck/handler.go:128","msg":"Health Check state change","status":"ready"}
{"level":"info","ts":1600365618.2731628,"caller":"app/server.go:232","msg":"Starting CMUX server","port":16686,"addr":":16686"}
{"level":"info","ts":1600365618.2732031,"caller":"app/server.go:208","msg":"Starting HTTP server","port":16686,"addr":":16686"}
{"level":"info","ts":1600365618.273346,"caller":"app/server.go:221","msg":"Starting GRPC server","port":16686,"addr":":16686"}


➜ mkdir ~/source/opentele
➜ cd opentele
➜ cat > jaeger.properties << EOF
JAEGER_ENDPOINT=http://jaeger-collector:14268/api/traces
JAEGER_REPORTER_LOG_SPANS=true\nJAEGER_SAMPLER_TYPE=const
JAEGER_SAMPLER_PARAM=1
JAEGER_PROPAGATION=b3
EOF
➜  opentele cat jaeger.properties
JAEGER_ENDPOINT=http://jaeger-collector:14268/api/traces
JAEGER_REPORTER_LOG_SPANS=true
JAEGER_SAMPLER_TYPE=const
JAEGER_SAMPLER_PARAM=1
JAEGER_PROPAGATION=b3


Create Node JS Express Simple App
 ➜ tutorial_dir=$(PWD)
mkdir nodejs-tracing
cd nodejs-tracing
appsody init kabanero/nodejs-express
➜  code .
➜  cat package.json
{
  "name": "nodejs-express-simple",
  "version": "1.0.0",
  "description": "Simple Node.js Express application",
  "license": "Apache-2.0",
  "main": "app.js",
  "repository": {
    "type": "git",
    "url": "https://github.com/appsody/stacks.git",
    "directory": "incubator/nodejs-express/templates/simple"
  },
  "scripts": {
    "test": "mocha"
  },
  "devDependencies": {
    "chai": "^4.2.0",
    "mocha": "^7.1.1",
    "request": "^2.88.0"
  }
}

➜   cat app.js
module.exports = (/*options*/) => {
  // Use options.server to access http.Server. Example with socket.io:
  //     const io = require('socket.io')(options.server)
  const app = require('express')()

  app.get('/', (req, res) => {
    // Use req.log (a `pino` instance) to log JSON:
    req.log.info({message: 'Hello from Appsody!'});
    res.send('Hello from Appsody!');
  });

  return app;
};

➜    cat ../jaeger.properties
➜   appsody run \
  --name "nodejs-tracing" \
  --publish "8083:8080" \
  --docker-options="--env-file ../jaeger.properties" 
 --network opentrace_network

➜   curl http://localhost:3000
Hello from Appsody!

Now running without Jaeger tracing code

Change package.json to rename app
    "name": "app-a-nodejs",
Change package.json to add jaeger dependency
"dependencies": {
    "jaeger-client": "^3.17.1"
  },
Change app.js to include tracing code…
const app = require('express')()
//
// Tutorial begin: Global initialization block
//
var initTracerFromEnv = require('jaeger-client').initTracerFromEnv;
var config = {
  serviceName: 'app-a-nodejs',
};
var options = {
};
var tracer = initTracerFromEnv(config, options);
//
// Tutorial end: Global initialization block
//
app.get('/', (req, res) => {
  //
  // Tutorial begin: OpenTracing new span
  //
  const span = tracer.startSpan('http_request');
  //
  // Tutorial end: OpenTracing new span
  //
// Use req.log (a pino instance) to log JSON:
  res.send('Hello from Appsody!');
//
  // Tutorial begin: Send span information to Jaeger
  //
  span.log({'event': 'request_end'});
  span.finish();
  //
  // Tutorial end: Send span information to Jaeger
  //
});
module.exports.app = app;

➜   appsody run \
  --name "nodejs-tracing" \
  --publish "8083:8080" \
  --docker-options="--env-file ../jaeger.properties" 
 --network opentrace_network

➜   curl http://localhost:3000
Hello from Appsody!

Lauch Jaeger UI to see the tracing log
http://localhost:16686
Select "Service": app-a-nodejs
Hit Find traces..

Create a Springboot App with Actuator included..


 ➜ mkdir springboot-tracing
 ➜ cd springboot-tracing
 ➜ appsody init kabanero/java-spring-boot2
➜ idea .
➜ appsody run \
  --name springboot-tracing \
  --docker-options="--env-file ../jaeger.properties" \
 --network opentrace_network
➜ docker ps
872d6a5240f3        kabanero/java-spring-boot2:0.3                "/.appsody/appsody-c…"   About a minute ago   Up 59 seconds       0.0.0.0:5005->5005/tcp, 0.0.0.0:8080->8080/tcp, 0.0.0.0:8443->8443/tcp, 0.0.0.0:35729->35729/tcp

Cat Main.java
package application;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Main {
	public static void main(String[] args) {
                SpringApplication.run(Main.class, args);
}
}


➜ curl http://localhost:8080/actuator
You can also openbrowser and navigate to http://localhost:8080/

The springboot app is now running without Jaeger tracing
Enable the app with Jaeger Tracing

Change pom.xml 
To include appname
    <artifactId>app-b-springboot</artifactId>


Change Main.java
package application;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
//
// Tutorial: Begin import statements for Jaeger and OpenTracing
//
import org.springframework.context.annotation.Bean;
import io.jaegertracing.Configuration;
import io.opentracing.Tracer;
//
// Tutorial: End import statements for Jaeger and OpenTracing
//
@SpringBootApplication
public class Main {
//
        // Tutorial: Begin initialization of OpenTracing tracer
        //
        @Bean
        public Tracer initTracer() {
          return Configuration.fromEnv("app-b-springboot").getTracer();
        }
        //
        // Tutorial: End initialization of OpenTracing tracer
        //
public static void main(String[] args) {
                SpringApplication.run(Main.class, args);
        }
}

Now stop and relaunch the Springboot app with Jaeger instrumentation

➜ appsody run \
  --name springboot-tracing \
  --docker-options="--env-file ../jaeger.properties" \
 --network opentrace_network

➜ curl http://localhost:8080
➜  inferno git:(development) curl http://localhost:8080
<!DOCTYPE html>
<html>
  <head>
    <title>Hello from Appsody!</title>
  </head>
  <body>
    <h3>Hello from Appsody!</h3>
    <p>Next steps with Spring Boot 2:
      <ul>
        <li><a target="_blank" rel="noopener" href="https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/">Spring Boot Reference Guide</a></li>
        <li><a target="_blank" rel="noopener" href="https://spring.io/guides">Spring Guides</a></li>
        <li><a target="_blank" rel="noopener" href="/actuator">Spring Boot Actuator endpoints</a>:
          <ul>
            <li><a target="_blank" rel="noopener" href="/actuator/health">Health</a></li>
            <li><a target="_blank" rel="noopener" href="/actuator/liveness">Liveness</a></li>
            <li><a target="_blank" rel="noopener" href="/actuator/metrics">Metrics</a></li>
            <li><a target="_blank" rel="noopener" href="/actuator/prometheus">Prometheus</a></li>
          </ul>
        </li>
      </ul>
    </p>
  </body>
</html>
➜
➜  ~ curl http://localhost:8080/actuator
{"_links":{"self":{"href":"http://localhost:8080/actuator","templated":false},"liveness":{"href":"http://localhost:8080/actuator/liveness","templated":false},"health":{"href":"http://localhost:8080/actuator/health","templated":false},"health-component":{"href":"http://localhost:8080/actuator/health/{component}","templated":true},"health-component-instance":{"href":"http://localhost:8080/actuator/health/{component}/{instance}","templated":true},"prometheus":{"href":"http://localhost:8080/actuator/prometheus","templated":false},"metrics":{"href":"http://localhost:8080/actuator/metrics","templated":false},"metrics-requiredMetricName":{"href":"http://localhost:8080/actuator/metrics/{requiredMetricName}","templated":true}}}%

Navigate to Jaeger UI 
http://localhost:16686
Select "Service": app-b-springboot
Hit Find traces..

Next Add the instrumentation and code to Call SpringBoot API from NodeJs Express 

Change Node JS App 

Package.json;  Add dependencies
"dependencies": {
    "jaeger-client": "^3.17.1",
    "opentracing": "latest",
    "request-promise": "^4.2.0",
    "uuid-random": "latest"
  },

➜  nodejs-tracing cat serviceBroker.js
// serviceBroker.js
// ================

const request = require('request-promise');
const { Tags, FORMAT_HTTP_HEADERS } = require('opentracing');

var serviceTransaction = function(serviceCUrl, servicePayload, parentSpan) {
    const tracer = parentSpan.tracer();
    const span = tracer.startSpan("service", {childOf: parentSpan.context()});
    var callResult = callService(serviceCUrl, servicePayload, span)
        .then( data => {
            span.setTag(Tags.HTTP_STATUS_CODE, 200)
            span.finish();
        })
        .catch( err => {
            console.log(err);
            span.setTag(Tags.ERROR, true)
            span.setTag(Tags.HTTP_STATUS_CODE, err.statusCode || 500);
            span.finish();
        });
    return callResult;
}

async function callService(serviceCUrl, servicePayload, parentSpan) {
    const tracer = parentSpan.tracer();
    const url = serviceCUrl;
    const body = servicePayload

    const span = parentSpan;
    const method = 'GET';
    const headers = {};
    span.setTag(Tags.HTTP_URL, serviceCUrl);
    span.setTag(Tags.SPAN_KIND, Tags.SPAN_KIND_RPC_CLIENT);
    span.setBaggageItem("baggage", true);
    tracer.inject(span, FORMAT_HTTP_HEADERS, headers);

    var serviceCallOptions = {
        uri: url,
        json: true,
        headers: headers,
        body: servicePayload
    };
    try {
        const data = await request(serviceCallOptions);
        span.finish();
        return data;
    }
    catch (e) {
        span.setTag(Tags.ERROR, true)
        span.finish();
        throw e;
    }

}

module.exports = serviceTransaction;


➜  nodejs-tracing cat app.js
module.exports = (/*options*/) => {
  // Use options.server to access http.Server. Example with socket.io:
  //     const io = require('socket.io')(options.server)

const app = require('express')()

//
// Tutorial begin: Remote requests to other applications
//
const uuid = require('uuid-random')
const serviceTransaction = require('./serviceBroker.js')
//
// Tutorial end: Remote requests to other applications
//

//
// Tutorial begin: OpenTracing initialization
//
const opentracing = require('opentracing')
var initTracerFromEnv = require('jaeger-client').initTracerFromEnv
var config = { serviceName: 'app-a-nodejs' }
var options = {}
var tracer = initTracerFromEnv(config, options)

// This block is required for compatibility with the service meshes
// using B3 headers for header propagation
// https://github.com/openzipkin/b3-propagation
const ZipkinB3TextMapCodec = require('jaeger-client').ZipkinB3TextMapCodec
let codec = new ZipkinB3TextMapCodec({ urlEncoding: true });
tracer.registerInjector(opentracing.FORMAT_HTTP_HEADERS, codec);
tracer.registerExtractor(opentracing.FORMAT_HTTP_HEADERS, codec);

opentracing.initGlobalTracer(tracer)
//
// Tutorial end: OpenTracing initialization
//

app.get('/', (req, res) => {
  res.send('Hello from Appsody Binu1!')
})

// Tutorial begin: Transaction A-B
app.get('/node-springboot', (req, res) => {
  const baseUrl = 'http://springboot-tracing:8080'
  const serviceCUrl = baseUrl + '/resource'
  const spanName = 'http_request_ab'

  // https://opentracing-javascript.surge.sh/classes/tracer.html#extract
  const wireCtx = tracer.extract(opentracing.FORMAT_HTTP_HEADERS, req.headers)
  const span = tracer.startSpan(spanName, { childOf: wireCtx })
  span.log({ event: 'request_received' })

  const payload = { itemId: uuid(),
                    count: Math.floor(1 + Math.random() * 10)}
  serviceTransaction(serviceCUrl, payload, span)
    .then(() => {
      const finishSpan = () => {
        span.log({ event : 'request_end'})
        span.finish()
      }

      res.on('finish', finishSpan)

      res.send(payload)
    })
})
// Tutorial end: Transaction A-B

// Tutorial begin: Transaction A-C
/*
app.get('/node-jee', (req, res) => {
  const baseUrl = 'http://jee-tracing:9080'
  const serviceCUrl = baseUrl + '/starter/service'
  const spanName = 'http_request_ac'

  const wireCtx = tracer.extract(opentracing.FORMAT_HTTP_HEADERS, req.headers)
  const span = tracer.startSpan(spanName, { childOf: wireCtx })
  span.log({ event: 'request_received' })

  const payload = { order: uuid(),
                    total: Math.floor(1 + Math.random() * 10000)}
  serviceTransaction(serviceCUrl, payload, span)
    .then(() => {
      const finishSpan = () => {
        span.log({ event : 'request_end'})
        span.finish()
      }

      res.on('finish', finishSpan)

      res.send(payload)
    })
})
*/
// Tutorial end: Transaction A-C

module.exports.app = app;

return app;
};
➜  nodejs-tracing



➜  nodejs-tracing appsody run \\n  --name "nodejs-tracing" \\n  --publish "8083:8080" \\n  --docker-options="--env-file ../jaeger.properties" \\n  --network opentrace_network'

➜ curl http://localhost:3000/node-springboot








