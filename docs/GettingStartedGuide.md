# Quick Start Guide
This guide provides an overview of the main resources for the Comms Router API, introduces the prerequisites and assists with enabling first application build.

**Note:** the commands listed below are designed to be used with a Unix shell. Some of them need to be modified in order to work on Windows.

##Process to get up and running
1. Setting up Voice
2. Creating a router
4. Creating queues
4. Creating plans
5. Creating agents
6. Creating tasks

## Supporting documentation
* Predicate [expression guide](blob/master/docs/ExpressionSyntax.md) for plans, queues and agents.
* OpenAPI spec http://localhost:8080/comms-router-web/swagger-ui.html
* Interactive Voice Response (IVR) guide https://developer.nexmo.com/voice/voice-api/guides/interactive-voice-response

### Prerequisites
* Setup a [Nexmo account](https://dashboard.nexmo.com/sign-up)
* Rent a virtual number using [Dashboard](https://dashboard.nexmo.com/buy-numbers) or [Developer API](https://developer.nexmo.com/api/developer/numbers) and set the webhook endpoint to your app

## 1. Setting up Voice
An overview and Getting started guide for Voice can be found here https://developer.nexmo.com/voice/voice-api/overview

* [Create an application](https://developer.nexmo.com/concepts/guides/applications#apps_quickstart) and associate it with your virtual number. Nexmo retrieves the initial NCCO from the answer_url webhook and sends the dtmf input to the eventUrl webhook defined in the initial NCCO
* Create your NCCO using the IVR use case with the required task and agent capabilities (link to demo app) link it to your application (answer_url)
* Create client application to manage Comms Router and Voice or install and use the [Comms Router demo application](/tree/master/demo-application)

## 2. Create a router
When creating a router you can either specify the router `ref` ID for example `MY_ROUTER` or leave blank and it will be automatically generated. All other resources are identified by the their `ref` ID. When creating a router the `ref` ID must be unique.
```
curl -X PUT http://localhost:8080/comms-router-web/api/routers/MY_ROUTER
```

## 3. Create queues
```
curl -X PUT http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/queues/MY_QUEUE \
  -H 'Content-Type:application/json' \
  -d$'{"predicate":"HAS(#{language},\'en\')"}}'  
```
**Note:** `-d$ ... }` is used to escape the JSON within the `predicate` key value pair for the cURL command.

## 4. Create plans
The following plan called `MY_PLAN` has a description and rules which include tasks must have `language == en` and will be routed to `MY_QUEUE` with a priority and timeout set. If no agents are available in this queue or if tineout limit reached with no other queues the task will move to the `DEFAULT_QUEUE`.

In the example below make sure you have a queue created otherwise it is possible to create a plan without queue specifications. Let's create a plan with a rule that will route tasks requiring Spanish agent in our Spanish queue. Tasks that don't match this rule we will route to the English queue.
```
curl -X PUT http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/plans/MY_PLAN \
  -H 'Content-Type:application/json' \
  -d$'{"description":"Description of MY_PLAN", "rules":[{"tag":"english", "predicate":"#{language} == \'en\'", "routes":[{"queueRef":"MY_QUEUE", "priority":3, "timeout":300}]}], "defaultRoute":{"queueRef":"MY_QUEUE", "priority":0, "timeout":0}}}'
```

## 5. Create agents
Agent can be created using a defined `ref` or allow this to be automatically created by the service. The example below creates a new user called `ALEX` who has a SIP URI and English `en` language capability. When creating an agent the `ref` ID must be unique.
```
curl -X PUT http://localhost:8080/comms-router-web/api/MY_ROUTER/demo/agents/ALEX \
  -H 'Content-Type:application/json' \
  -d '{"address":"sip:alex@vonage.com","capabilities":{"language":["en"]}}'
```
If not specified an agent's status will be set to `offline` by default. Other statuses include `offline`, `ready`, `busy`, `unavailable`.

### List agents
#### Request
```
curl -X GET http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/agents
```

#### Response
```
[
  {
    "ref": "alex",
    "routerRef": "MY_ROUTER",
    "capabilities": {
      "language": "en"
    },
    "address": "sip:alex@vonage.com",
    "state": "offline",
    "queueIds": [
      "queue-en"
    ]
  },
  {
    ...
  }
]
```

## 6. Create tasks
Below is a request to create a task `MY_TASK` that requires an agent to be able to speak English. We assign to it the plan `MY_PLAN` using `planRef` parameter. The `callbackUrl` parameter specifies the user application entry point to be called by the router for activity related with this task.
```
curl -X PUT http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/tasks/MY_TASK \
  -H 'Content-Type:application/json' \
  -d$'{"requirements":{"language":"en"},"planRef":"MY_PLAN","callbackUrl":"http://webhook.site/#/fae3ff2f-a1f7-4648-9804-21666b3bb15d"}'
```

In addition to using a plan to route tasks, the router accepts direct queue assignment by the user application.

```
curl -X PUT http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/tasks/MY_TASK \
  -H 'Content-Type:application/json' \
  -d$'{"queueRef":"MY_QUEUE","callbackUrl":"http://webhook.site/#/fae3ff2f-a1f7-4648-9804-21666b3bb15d"}'
```

### List tasks
#### Request
```
curl -X GET http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/tasks
```

#### Response
```
[
  {
    "ref": "MY_TASK",
    "routerRef": "MY_ROUTER",
    "requirements": {
      "language": "en"
    },
    "userContext": null,
    "state": "waiting",
    "planRef": null,
    "queueRef": "MY_QUEUE",
    "agentRef": null,
    "callbackUrl": "http://webhook.site/#/fae3ff2f-a1f7-4648-9804-21666b3bb15d"
  },
  {
    ...
  }
]
```

By default all tasks are in state `waiting` if all agents are in state `offline`. To change an agents status use the following command:
```
curl -X POST http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/agents/ALEX \
  -H 'Content-Type:application/json' \
  -d '{"state":"ready"}'
```
Task statuses are managed by the Comms Router and include: `waiting`, `canceled`, `assigned`, `completed`.

## Task flow
Once the router assigns a task an agent the agent status changes to `busy`. A call to the provided `callbackUrl` can be observed in `http://webhook.site/#/fae3ff2f-a1f7-4648-9804-21666b3bb15d`. 

When the user application is done with processing a task it must declare it as done:
```
curl -X POST http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/tasks/MY_TASK 
  -H 'Content-Type:application/json' 
  -d '{"state":"completed"}'
```

The router then releases the agent and they are available (`ready`) for other tasks. In this example the agent `ALEX` can serve more than one queue for example `EN_QUEUE` and `FR_QUEUE`, it will automatically get the other task we created.

```
curl http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/tasks
```

```
[
  {
    "ref": "EN_TASK",
    "routerRef": "MY_ROUTER",
    "requirements": {
      "language": "en"
    },
    "userContext": null,
    "state": "completed",
    "planId": null,
    "queueRef": "EN_QUEUE",
    "agentRef": null,
    "callbackUrl": "http://webhook.site/#/fae3ff2f-a1f7-4648-9804-21666b3bb15d"
  },
  {
    "ref": "FR_TASK",
    "routerRef": "MY_ROUTER",
    "requirements": null,
    "userContext": null,
    "state": "assigned",
    "planRef": null,
    "queueRef": "FR_QUEUE",
    "agentRef": "ALEX",
    "callbackUrl": "http://webhook.site/#/fae3ff2f-a1f7-4648-9804-21666b3bb15d"
  }
]
```

To clean up the flow we should finish by making the task complete.

```
curl -X POST http://localhost:8080/comms-router-web/api/routers/MY_ROUTER/tasks/EN_TASK \
  -H 'Content-Type:application/json'
  -d '{"state":"completed"}'
```
