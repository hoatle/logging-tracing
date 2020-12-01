# logging-tracing

logging and tracing libraries for cloud native apps. This library uses:
- winston for logging
- opentelemetry for tracing and monitoring metrics

We add tracing context into logging when it's available so that we can monitor apps better.

The target supported backends:
- Google Cloud Tracing and Logging
- Grafana
- EFK stack (Elasticsearch, Fluentd and Kibana)
- Amazon CloudWatch


## How to use

- npm install:

```bash
$ npm install teracyhq-incubator/logging-tracing#develop
```


### Tracing

- Create tracing.js file with the following similar content:

```js
const { initTracing } = require('logging-tracing');
// for google cloud tracing
const { CloudPropagator } = require('@google-cloud/opentelemetry-cloud-trace-propagator');


const exporterSpec = process.env.TRACE_EXPORTER || "";

let exporterOpts = {}, registerOpts = {};

switch(exporterSpec) {
  case "ZIPKIN":
    exporterOpts = Object.assign(exporterOpts, tracing.getExporterOptions(), {
      serviceName: "logging-tracing-app"
    });
    break;
  case "GOOGLE_CLOUD_TRACE":
    registerOpts = {
      // Use CloudPropagator
      propagator: new CloudPropagator()
    };
    break;
}


const opts = {
  provider: {
    plugins: {
      http: {
        enabled: true,
        path: '@opentelemetry/plugin-http'
        // ignoreIncomingPaths: ['/healthz']
      },
      https: {
        enabled: true,
        path: '@opentelemetry/plugin-https'
      },
      express: {
        enabled: true,
        path: '@opentelemetry/plugin-express'
      }
    }
  },
  exporter: exporterOpts,
  register: registerOpts
}

initTracing(opts)
```


### Logging

- Follow the similar content below:

```js
const { getLogger } = require('logging-tracing');
const logger = getLogger('my category');
logger.info('information message'); // etc
// see: https://github.com/winstonjs/winston

// Set logging level by env var: LOGGING_LEVEL=debug|info|notice|...

// using the logging methods/ levels below:
// { 
//   emerg: 0,   // One or more systems are unusable.
//   alert: 1,   // A person must take an action immediately.
//   crit: 2,    // Critical events cause more severe problems or outages.
//   error: 3,   // Error events are likely to cause problems.
//   warning: 4, // Warning events might cause problems.
//   notice: 5,  // Normal but significant events, such as start up, shut down, or a configuration change
//   info: 6,    // Routine information, such as ongoing status or performance.
//   debug: 7    // Debug or trace information
// }
// see: https://github.com/winstonjs/triple-beam/blob/master/config/syslog.js
```


### Metrics

//TODO(hoatle):

