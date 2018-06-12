# Introduction to Comms Router
The Nexmo Comms Router API enables businesses to leverage self-hosted or on premises APIs to manage Contact Centre task management. The modern Contact Center is designed to be flexible and extensible through allowing developers to customise integrations and leverage Nexmo Comms Router API as a key building block.

As part of the Contact Center building blocks architecture this documentation also includes:
* A [Demo Application](demo-application/README.md) to show how to apply the API in a Contact Center use case.
* A [Graphical User Interface Application](gui-application/README.md) to enable admins to manage agents.

# Pre-requisites
* Nexmo [Account](https://dashboard.nexmo.com/sign-up)
* Nexmo [Phone Numbers](https://developer.nexmo.com/account/guides/numbers)
* Nexmo [Voice API](https://developer.nexmo.com/voice/voice-api/overview)
* Nexmo Comms Router API

# Key Concepts
* Customer - User that is initating the contact creaing the task that is then routed to an agent through a queue.
* Router - The parent container to join all of the router entities tasks, agents, queues and plans.
* Plans - Construct that manages multiple Queues, priority of tasks in against queues and failover if a queue has no available Agents to receive a task.
* Queues - A bucket to collect tasks waiting for the next available agent. The Task is routed to a queue based on conditional logic (predicate) predefined by the customer requirements for example if they require to be routed to speak to an agent with a specific skill like language.
* Agents - An endpoint or phyiscal agent able to handle tasks, characterized by its capabilities or assigned skills it has based on key value pairs set.
* Capabilities - Conditional logic assigned to agents and queues to route tasks. These are made of key value pairs or arrays, single or multiple value conditions
* Tasks - A work item characterized by its capabilities for example a set of skills an Agent requires to have a task routed to them based on key value pairs set.
  
# Features
* Agent capabilities can by text, numeric integars or ranges, boolean key value pairs or an array of strings.
* Agent availability can be set to `offline`, `ready`, `busy`, `unavailable` and is updated to `busy` based on if a task has been assigned to them.
* Within the plans it is possible to manage priority order of queues, priority of a task and timeout of a task to enable failover to another queue.

# Customer Journey Flows

## Callback Voice Call (PSTN):
1. Customer submits an online callback request form
2. Form submitted to server application
3. Task is created with routing requirements from Customer request
4. Available Agent is found with matching skills
5. Agent accepts reservation and dials customer phone number (PSTN)
6. Call is initiated and connects the Agent to the Customer

![Callback a Customer using Voice over PSTN](to_do.png)

## Inbound Voice Call (PSTN):
1. Customer calls Nexmo phone number
2. Client receives request via Webhook
3. Predefined IVR option selected by Customer
4. Client receives IVR option from Voice API
5. Comms Router task created from client
6. Available agent found with matching criteria
7. Agent accepts request
8. Customer connected to agent

![Customer Journey Inbound Call](to_do.png)

## Agent Management and Real-Time Tasks Dashboard:
Manage agents availability, phone numbers, skills, queues and plans within a Dashboard application. View tasks associated with queues in real-time and manage skills of agents and their availbility.

![Agent Management Dashboard](to_do.png)

# Installation
Before installation, you’ll need the following from the [Nexmo Dashboard](https://dashboard.nexmo.com/sign-in). If you need to create an account [Sign up for a Nexmo](https://dashboard.nexmo.com/sign-up).

* Your Nexmo API key and API secret which can be found on the Dashboard: https://dashboard.nexmo.com/settings
* A Nexmo phone number, either [purchase one](https://dashboard.nexmo.com/buy-numbers) or you can use a number you have (already purchased)[https://dashboard.nexmo.com/your-numbers].
* Create a [Nexmo Call Control Object](https://developer.nexmo.com/voice/voice-api/guides/ncco) (NCCO) with capabilities mapping to the IVR.
* Enable a webhook for capturing events from your Voice Application during calls (this is optional).
* Create a [Nexmo Voice Application](https://dashboard.nexmo.com/voice/create-application) to link you number, NCCO and webhook.

**Note:** It is recommended that you have an upgraded Nexmo account with credit before installation.

## Requirements
* Java - Oracle JDK/JRE 8 (build/runtime)
* Apache Maven - 3.5 (build)
* SQL Server - MySQL 5.7 (runtime)
* Java Servlet Container - Tomcat 8 (runtime)

The Comms Router API may work with different types of Java, SQL Server or Web Container and is currently being tested and maintained with the above component versions.

## Build
* Install Java and Maven, clone the repo and execute: `mvn install`
* The resulting war file should be: `web/target/comms-router-web.war`

## Install
* Create database for the router.
* Create database user to be used by the router and grant this user access to the newly created database.
* Configure JNDI data source with name `jdbc/commsRouterDB`.

Read more information on [how to do set up Tomcat](docs/ConfiguringDatabaseAccess.md).

* Deploy `comms-router-web.war` into Tomcat.
* Depending on your Tomcat settings this can be done by simple copying it to the Tomcat's webapps directory.
* List routers: `curl http://localhost:8080/comms-router-web/api/routers`

# Additional resources

  * Browse [docs](docs) directory for additional documentation.
  * See our [demo application](demo-application/README.md) as an example how to integrate the router with Nexmo API]






# Quick Start Guide







## Quick tutorial

Note that the commands listed below are meant to be used with a Unix shell. Some of them need to be modified in order to work on Windows.

For example, the command for creating queues would look like this:

`
curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/queues/queue-es -H "Content-Type:application/json" -d "{\"predicate\":\"HAS(#{language},'es')\"}}"
`

For more information take a look at [this](winComms.bat) batch file.

**Resource Identification**

Routers are identified by their Ref ID, which can be provided by the user or generated by the system. Router Ref IDs must be unique within the system.

All other resources are identified by the their Ref ID, again provided by the user or generated by the application and the Ref ID of the router that contains them. Their Ref IDs must be unique within their containing router.

**Create a router**, providing it's Ref ID:

`curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router`

Or request the system to assign the Ref ID:

`curl -X POST http://localhost:8080/comms-router-web/api/routers`

The Ref ID is returned in the response body:

`{"ref":"HaOYogXa8qgX9NlRHJi9Y2"}`

And URL of the created entity is in the header LOCATION of the response:

`Location: http://localhost:8080/comms-router-web/api/routers/HaOYogXa8qgX9NlRHJi9Y2`

**Create some queues.**

For example, let's create one handled by English speaking agents:

`
curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/queues/queue-en -H 'Content-Type:application/json' -d$'{"predicate":"HAS(#{language},\'en\')"}}'
`

And one more handled by these speaking Spanish:

`
curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/queues/queue-es -H 'Content-Type:application/json' -d$'{"predicate":"HAS(#{language},\'es\')"}}'
`

**Create agents.**

Let's assume we have have three agents - Alice speaking English, Juan speaking Spanish and Maria speaking both English and Spanish.

So for Alice we'll have:

`
curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/agents/alice -H 'Content-Type:application/json' -d'{"address":"sip:alice@comms-router.org","capabilities":{"language":["en"]}}'
`

for Juan:

`
curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/agents/juan -H 'Content-Type:application/json' -d '{"address":"sip:juan@comms-router.org","capabilities":{"language":["es"]}}'
`

and for Maria:

`
curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/agents/maria -H 'Content-Type:application/json' -d'{"address":"sip:maria@comms-router.org","capabilities":{"language":["en","es"]}}'
`

Note the _address_ field. It does not affect the router logic and is used by the user application to store information that it needs to route tasks to this agent. In our example addresses are SIP URIs.

**List agents and note the queue assignments:**

`curl http://localhost:8080/comms-router-web/api/routers/my-router/agents`

```
[
  {
    "ref": "alice",
    "routerRef": "my-router",
    "capabilities": {
      "language": "en"
    },
    "address": "sip:alice@comms-router.org",
    "state": "offline",
    "queueIds": [
      "queue-en"
    ]
  },
  {
    "ref": "juan",
    "routerRef": "my-router",
    "capabilities": {
      "language": "es"
    },
    "address": "sip:juan@comms-router.org",
    "state": "offline",
    "queueIds": [
      "queue-es"
    ]
  },
  {
    "ref": "maria",
    "routerRef": "my-router",
    "capabilities": {
      "language": [
        "es",
        "en"
      ]
    },
    "address": "sip:maria@comms-router.org",
    "state": "offline",
    "queueIds": [
      "queue-es",
      "queue-en"
    ]
  }
]
```

**Create a plan.**

Let's create a plan with a rule that will route tasks requiring Spanish agent in our Spanish queue. Tasks that don't match this rule we will route to the English queue.

`curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/plans/by-language -H 'Content-Type:application/json' -d$'{"description":"put your plan description", "rules":[{"tag":"spanish", "predicate":"#{language} == \'es\'", "routes":[{"queueRef":"queue-es", "priority":3, "timeout":300}, {"priority":10, "timeout":800}]}], "defaultRoute":{"queueRef":"queue-en"}}'`

**Create tasks.**

`curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/tasks/task-es -H 'Content-Type:application/json' -d$'{"requirements":{"language":"es"},"planRef":"by-language","callbackUrl":"https://requestb.in/1koh4zk1?inspect"}'`

Here the task requires agent speaking Spanish and we assign to it our plan that routes tasks by language.

The "callbackUrl" parameter specifies the user application entry point to be called by the router for activity related with this task. An easy way to test the router is to use a requestb.in to accept the callback, as we are doing in this example.

In addition to using a plan to route tasks, the router accepts direct queue assignment by the user application:

`curl -X PUT http://localhost:8080/comms-router-web/api/routers/my-router/tasks/task-en -H 'Content-Type:application/json' -d$'{"queueRef":"queue-en","callbackUrl":"https://requestb.in/1koh4zk1?inspect"}'`

**Let's list the tasks and see the queues assigned:**

`curl http://localhost:8080/comms-router-web/api/routers/my-router/tasks`

```
[
  {
    "ref": "task-es",
    "routerRef": "my-router",
    "requirements": {
      "language": "es"
    },
    "userContext": null,
    "state": "waiting",
    "planRef": null,
    "queueRef": "queue-es",
    "agentRef": null,
    "callbackUrl": "https://requestb.in/1koh4zk1"
  },
  {
    "ref": "task-en",
    "routerRef": "my-router",
    "requirements": null,
    "userContext": null,
    "state": "waiting",
    "planRef": null,
    "queueRef": "queue-en",
    "agentRef": null,
    "callbackUrl": "https://requestb.in/1koh4zk1?inspect"
  }
]
```

All tasks are in state "waiting" as all our agents are in state "offline".

**Change agent's state.**

`curl -X POST http://localhost:8080/comms-router-web/api/routers/my-router/agents/maria -H 'Content-Type:application/json' -d '{"state":"ready"}'`

Now the router assigns a task this agent and changes its state to "busy". Call to the provided callbackUrl can be observed in requestb.in.

**Complete Task.**

When the user application is done with processing a task it must declare it as done:

`curl -X POST http://localhost:8080/comms-router-web/api/routers/my-router/tasks/task-es -H 'Content-Type:application/json' -d '{"state":"completed"}'`

The router then releases the agent and it is available for other tasks. As in this example the agent "Maria" can serve both queues, it will automatically get the other task we created:

`curl http://localhost:8080/comms-router-web/api/routers/my-router/tasks`

```
[
  {
    "ref": "task-es",
    "routerRef": "my-router",
    "requirements": {
      "language": "es"
    },
    "userContext": null,
    "state": "completed",
    "planId": null,
    "queueRef": "queue-es",
    "agentRef": null,
    "callbackUrl": "https://requestb.in/1koh4zk1"
  },
  {
    "ref": "task-en",
    "routerRef": "my-router",
    "requirements": null,
    "userContext": null,
    "state": "assigned",
    "planRef": null,
    "queueRef": "queue-en",
    "agentRef": "maria",
    "callbackUrl": "https://requestb.in/1koh4zk1"
  }
]
```

We should finish our journey by making this task completed:

`curl -X POST http://localhost:8080/comms-router-web/api/routers/my-router/tasks/task-en -H 'Content-Type:application/json' -d '{"state":"completed"}'`