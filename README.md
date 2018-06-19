# Introduction to Comms Router
The Nexmo Comms Router API enables businesses to leverage self-hosted or on premises APIs to manage Contact Centre task management. The modern Contact Center is designed to be flexible and extensible through allowing developers to customise integrations and leverage Nexmo Comms Router API as a key building block.

As part of the Contact Center building blocks architecture this documentation also includes:
* A [Demo Application](demo-application/README.md) to show how to apply the API in a Contact Center use case.
* A [Graphical User Interface Application](gui-application/README.md) to enable admins to manage agents.

## Pre-requisites
* Nexmo Account https://dashboard.nexmo.com/sign-up
* Nexmo Phone Numbers https://developer.nexmo.com/account/guides/numbers
* Nexmo Voice API https://developer.nexmo.com/voice/voice-api/overview
* Nexmo Comms Router API installed

## Key Concepts
* *Router* - The parent container that joins all of the router entities tasks, agents, queues and plans.
* *Plans* - Container that manages multiple queues, priority of tasks in queues and failover if a queue has no available agents.
* *Queues* - A bucket to collect tasks waiting for the next available agent. The task is routed to a queue based on conditional logic (predicate) predefined by the customer requirements for example if they require to be routed to speak to an agent with a specific skill like language.
* *Agents* - An endpoint for a phyiscal agent able to handle tasks, characterized by its capabilities or assigned skills it has based on key value pairs set.
* *Capabilities* - Conditional logic assigned to agents and queues to route tasks. These are made of key value pairs or arrays, single or multiple value conditions.
* *Tasks* - A work item characterized by its capabilities for example a set of skills an Agent requires to have a task routed to them based on key value pairs set.
* *Customer* - End user that is initating the contact request and creating the task that is then routed to an agent through a queue.
  
## Features
* Agent capabilities can be text, numeric integars or ranges, boolean key value pairs or an array of strings.
* Agent availability can be set to `offline`, `ready`, `busy`, `unavailable` and is updated to `busy` based on if a Task has been assigned to them.
* Within the Plans it is possible to manage priority order of Queues, priority of a Task and timeout of a Task to enable failover to another Queue.

# Customer Journey Flows

## Callback Voice Call (PSTN):
1. Customer submits an online callback request form
2. Form submitted to server application
3. Task is created with routing requirements from customer request
4. Available Agent is found with matching skills
5. Agent accepts reservation and dials customer phone number (PSTN)
6. Call is initiated and connects the Agent to the customer

![Nexmo Comms Router callback using Voice over PSTN customer journey](to_do.png)

## Inbound Voice Call (PSTN):
1. Customer calls Nexmo phone number
2. Client receives request via Webhook
3. Predefined IVR option selected by the customer
4. Client receives IVR option from Voice API
5. Comms Router Task created from client
6. Available Agent found with matching criteria
7. Agent accepts request
8. Customer connected to Agent

![Nexmo Comms Router inbound call customer journey](to_do.png)

## Agent Management and Real-Time Tasks Dashboard:
Manage Agents availability, phone numbers, Skills, Queues and Plans within a Dashboard application. View tasks associated with Queues in real-time and manage Skills of Agents and their availbility.

![Agent Management Dashboard](to_do.png)

# Installation
Before installation, you’ll need the following from the [Nexmo Dashboard](https://dashboard.nexmo.com/sign-in). If you need to create an account [sign up for a Nexmo](https://dashboard.nexmo.com/sign-up).

* Your [Nexmo API key and API](https://dashboard.nexmo.com/settings) secret which can be found on the Dashboard.
* A Nexmo phone number, either [purchase one](https://dashboard.nexmo.com/buy-numbers) or you can use a number you have [already purchased](https://dashboard.nexmo.com/your-numbers).
* Create a [Nexmo Call Control Object](https://developer.nexmo.com/voice/voice-api/guides/ncco) (NCCO) with capabilities mapping to the IVR.
* Enable a webhook for capturing events from your Voice Application during calls (this is optional).
* Create a [Nexmo Voice Application](https://dashboard.nexmo.com/voice/create-application) to link you number, NCCO and webhook.

**Note:** It is recommended that you have an upgraded Nexmo account with credit before installation.

## System requirements
* Java - Oracle JDK/JRE 8 (build/runtime)
* Apache Maven - 3.5 (build)
* SQL Server - MySQL 5.7 (runtime)
* Java Servlet Container - Tomcat 8 (runtime)

The Comms Router API may work with different types of Java, SQL Server or Web Container and is currently being tested and maintained with the above component versions.

## Build
* Install Java and Maven, clone the repo and execute: `mvn install`
* The resulting war file should be: `web/target/comms-router-web.war`

## Install
* Create [database for the router](docs/ConfiguringDatabaseAccess.md).
* Create database user to be used by the router and grant this user access to the newly created database.
* Configure JNDI data source with name `jdbc/commsRouterDB`.

Read more information on [how to do set up Tomcat](docs/ConfiguringDatabaseAccess.md) for Comms Router.

* Deploy `comms-router-web.war` into Tomcat.
* Depending on your Tomcat settings this can be done by simple copying it to the Tomcat's webapps directory.
* List routers: `curl -X GET http://localhost:8080/comms-router-web/api/routers`

## Supporting documentation
* [Getting Started Guide](docs/GettingStartedGuide.md) for quick start.
* Predicate [expression guide](docs/ExpressionSyntax.md) for Agents and Queues.
* Set up of [database access and Tomcat configuration](docs/ConfiguringDatabaseAccess.md).
* How to [manage database migrations](docs/ManageDBMigrations.md).
* OpenAPI spec on localhost http://localhost:8080/comms-router-web/swagger-ui.html
