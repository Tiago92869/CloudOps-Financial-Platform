# CloudOps Financial Platform

## 1. Project Overview

**CloudOps Financial Platform** is a cloud-native internal operations platform designed to simulate the kind of distributed backend system used by financial, cloud, or enterprise infrastructure companies.

The platform allows users to create and manage sensitive operational requests such as financial-like operations, production access requests, deployment requests, and other controlled business operations. These operations may require validation, approval, auditing, notifications, and incident tracking.

The project is designed as a realistic portfolio project to practice backend, platform engineering, DevOps, observability, and deployment technologies commonly used in enterprise environments.

The main goal is not to build a simple CRUD application, but to build a distributed platform that demonstrates microservice architecture, asynchronous communication, centralized authentication, CI/CD, container orchestration, centralized logging, auditability, and operational monitoring.

---

## 2. Main Goals

The project should demonstrate the ability to design and implement a modern enterprise platform using:

* Java and Spring Boot for backend services
* Angular for the frontend application
* Keycloak for authentication and identity management
* Spring Security for service-level authorization
* Kafka for asynchronous communication between services
* PostgreSQL as the main relational database backend
* Elasticsearch for audit search and log search
* Fluentd for centralized log collection
* Kibana for log and audit visualization
* Docker for containerization
* Kubernetes for container orchestration
* OpenShift as an enterprise Kubernetes deployment target
* Jenkins for CI/CD automation
* GitHub for source control and Jenkins integration
* GitHub Container Registry or another registry for Docker images

---

## 3. High-Level Business Concept

The platform is used by different internal users inside a financial or cloud operations organization.

Users may create sensitive operations such as:

* Payment-like operation requests
* Production system access requests
* Sensitive data export requests
* Deployment requests
* Operational recovery requests

Each operation follows a controlled lifecycle. Depending on the operation type, risk level, requester role, and target system, the platform may automatically validate it, require approval, reject it, complete it, or create an incident.

A simplified example flow:

1. A user creates a sensitive operation.
2. The platform validates the operation.
3. If the operation is risky, approval is required.
4. A manager reviews and approves or rejects the request.
5. The operation status is updated.
6. Notifications are sent to the relevant users.
7. Every relevant action is stored as an audit event.
8. Logs are collected centrally.
9. Failures may create operational incidents.
10. Dashboards allow users to monitor the platform.

---

## 4. Target Users and Roles

The platform will support multiple user roles. Authentication and role management will be handled by Keycloak.

### Operator

An operator can create and track operations.

Example responsibilities:

* Create operation requests
* View own operations
* Cancel eligible operations
* Receive notifications

### Manager

A manager can review and approve/reject operations that require approval.

Example responsibilities:

* View pending approvals
* Approve operations
* Reject operations
* Add approval comments

### Auditor

An auditor can search and review audit records.

Example responsibilities:

* Search operation history
* Search user actions
* Search by correlation ID
* Review compliance-related activity

### Platform Engineer

A platform engineer can monitor incidents and platform health.

Example responsibilities:

* View operational incidents
* Resolve incidents
* Escalate incidents
* Check service health
* Review technical failures

### Administrator

An administrator has elevated privileges.

Example responsibilities:

* Manage application users and profiles
* Manage role assignments in coordination with Keycloak
* View all operations
* View all audit data
* Access platform administration features

---

## 5. High-Level Architecture

The platform will follow a microservice architecture.

At a high level:

```text
Angular Frontend
      |
      v
API Gateway
      |
      v
Java Spring Boot Microservices
      |
      v
Kafka Event Bus
      |
      v
PostgreSQL / Elasticsearch
      |
      v
Fluentd -> Elasticsearch -> Kibana
```

The Angular frontend communicates only with the API Gateway. Internal services are not exposed directly to the frontend.

The backend services communicate through a mix of synchronous HTTP calls and asynchronous Kafka events. Kafka should be used for workflow progression, notifications, auditing, incident creation, and other asynchronous processes.

---

## 6. Main Components

## 6.1 Angular Frontend

The Angular frontend is the main user interface of the platform.

It should provide pages for:

* Login
* Dashboard
* Operations
* Operation details
* Create operation
* Approvals
* Notifications
* Audit search
* Incidents
* Platform health
* User profile/settings

The frontend should use Keycloak/OpenID Connect for authentication.

Role-based UI behavior should be implemented so that users only see the sections relevant to their permissions.

Example:

* Operators see operations and notifications.
* Managers see approvals.
* Auditors see audit search.
* Platform engineers see incidents and platform health.
* Administrators see broader management options.

---

## 6.2 API Gateway

The API Gateway is the single public entry point for backend APIs.

Main responsibilities:

* Route frontend requests to the correct backend service
* Validate authentication tokens
* Apply route-level authorization rules
* Add or propagate correlation IDs
* Centralize request logging
* Hide internal service addresses from the frontend

The gateway should be implemented with Spring Cloud Gateway.

Example routes:

```text
/api/operations/**      -> operation-service
/api/approvals/**       -> approval-service
/api/notifications/**   -> notification-service
/api/audit/**           -> audit-service
/api/incidents/**       -> incident-service
/api/users/**           -> user-service
/api/platform-health/** -> platform-health-service or gateway aggregation endpoint
```

---

## 6.3 Keycloak

Keycloak will be used as the identity provider.

Keycloak responsibilities:

* User authentication
* Login page
* Token issuing
* Role management
* Group management
* Session management
* OpenID Connect support

The application should not implement custom password management.

Recommended Keycloak setup:

```text
Realm: cloudops

Clients:
- cloudops-ui
- api-gateway

Roles:
- OPERATOR
- MANAGER
- AUDITOR
- ADMIN
- PLATFORM_ENGINEER
```

---

## 6.4 User Service

The User Service will not manage passwords. Passwords and login credentials belong to Keycloak.

The User Service manages application-specific user profile data.

Example responsibilities:

* Store user profile linked to Keycloak user ID
* Store team or department information
* Store manager relationships
* Store notification preferences
* Provide user profile information to other services

Example data owned by this service:

```text
Application user ID
Keycloak user ID
Email
Display name
Team
Manager
Notification preferences
Application status
```

---

## 6.5 Operation Service

The Operation Service is the core business service of the platform.

It manages sensitive operations from creation to completion.

Example operation types:

* Payment-like operation
* Production access request
* Deployment request
* Sensitive data export request
* Recovery operation

Example operation statuses:

```text
CREATED
VALIDATING
WAITING_APPROVAL
APPROVED
REJECTED
PROCESSING
COMPLETED
FAILED
CANCELLED
```

Main responsibilities:

* Create operations
* Store operation state
* Publish operation events to Kafka
* React to validation and approval events
* Track operation lifecycle
* Expose operation status to the frontend

---

## 6.6 Validation/Risk Service

The Validation/Risk Service evaluates whether an operation is valid, risky, or requires approval.

Example checks:

* Is the requester allowed to create this operation?
* Is the target system sensitive?
* Is the operation high-value?
* Does the operation require manager approval?
* Is the operation duplicated?
* Should the operation be rejected immediately?

The service should mainly communicate through Kafka.

Example input event:

```text
operation.created
```

Example output events:

```text
operation.validated
operation.rejected
operation.requires_approval
```

---

## 6.7 Approval Service

The Approval Service manages approval workflows.

Main responsibilities:

* Create approval tasks
* List pending approvals
* Approve operations
* Reject operations
* Store approval comments
* Publish approval events

Example events:

```text
approval.requested
approval.approved
approval.rejected
```

The Approval Service should support simple approval workflows initially. Later, the project may add more advanced approval types such as two-person approval or security approval.

---

## 6.8 Notification Service

The Notification Service informs users about relevant platform events.

Examples:

* Operation created
* Operation approved
* Operation rejected
* Operation completed
* Operation failed
* Approval requested
* Incident created

The service should consume Kafka events and create user-facing notifications.

Initial implementation can use in-app notifications. Later versions may simulate email, Slack, or Teams notifications.

---

## 6.9 Audit Service

The Audit Service records important business and security events.

This service is critical for traceability and compliance.

It should consume events from Kafka and store searchable audit records in Elasticsearch.

Example audit records:

* User created operation
* Validation completed
* Approval requested
* Manager approved operation
* Manager rejected operation
* Operation completed
* Operation failed
* Incident created

The frontend should provide an audit search page where authorized users can search by:

* User
* Operation ID
* Event type
* Correlation ID
* Date range
* Service name

---

## 6.10 Incident Service

The Incident Service manages operational problems.

Incidents may be created manually or automatically when failures occur.

Example incident triggers:

* Operation processing fails repeatedly
* Kafka message cannot be processed
* Notification delivery fails
* Service health check fails
* Deployment fails

Example incident statuses:

```text
OPEN
ACKNOWLEDGED
ESCALATED
RESOLVED
CLOSED
```

Example severity levels:

```text
LOW
MEDIUM
HIGH
CRITICAL
```

---

## 7. Data Storage Strategy

## 7.1 PostgreSQL

PostgreSQL will be the main relational database backend for business data.

Recommended usage:

* User profiles
* Operations
* Approvals
* Notifications
* Incidents
* Service-specific relational data

Each microservice should ideally own its own database schema or database to respect service boundaries.

Initial development can use one PostgreSQL instance with separate databases or schemas per service.

Example:

```text
user_service_db
operation_service_db
approval_service_db
notification_service_db
incident_service_db
```

---

## 7.2 Elasticsearch

Elasticsearch should be used for search-oriented and log-oriented data.

Recommended usage:

* Audit events
* Centralized application logs
* Searchable operational history

Elasticsearch should not replace PostgreSQL for core transactional data.

The preferred approach is:

```text
PostgreSQL = source of truth for business state
Elasticsearch = search, audit exploration, log analysis, dashboards
```

---

## 7.3 Logs Backend Recommendation

For logs, Elasticsearch is a good backend for this project because it integrates naturally with Fluentd and Kibana.

Recommended log flow:

```text
Spring Boot services
      |
Container logs
      |
Fluentd
      |
Elasticsearch
      |
Kibana
```

This gives the project a realistic centralized logging stack.

Alternative options such as Loki + Grafana are also valid in modern systems, but since the target job explicitly mentions Elasticsearch, Fluentd, and Kibana, this project should use Elasticsearch as the log backend.

---

## 8. Kafka Event-Driven Communication

Kafka will be used as the event backbone of the platform.

The platform should use Kafka for asynchronous communication between services.

Example event categories:

```text
operation.created
operation.validated
operation.requires_approval
operation.approved
operation.rejected
operation.processing_started
operation.completed
operation.failed
approval.requested
approval.approved
approval.rejected
notification.created
audit.event.created
incident.created
```

Kafka will allow services to remain loosely coupled.

Example:

* Operation Service does not need to directly call Notification Service.
* It publishes an event.
* Notification Service consumes the event and creates a notification.
* Audit Service consumes the same event and stores an audit record.

This is one of the most important architectural concepts in the project.

---

## 9. Observability and Monitoring

The platform should include observability from the beginning.

Initial observability goals:

* Every request should have a correlation ID.
* Logs should be structured, preferably JSON.
* Logs should include service name, timestamp, level, correlation ID, and message.
* Service health should be exposed through Spring Boot Actuator.
* Logs should be collected by Fluentd.
* Logs should be searchable in Kibana.
* Audit events should be searchable in Elasticsearch.

Example Kibana dashboards:

* Errors by service
* Failed operations over time
* Pending approvals
* Incidents by severity
* Kafka consumer failures
* Logs by correlation ID
* Recent audit events

---

## 10. Deployment Strategy

The platform should support multiple deployment levels.

## 10.1 Local Development

Local development should use Docker Compose.

Docker Compose should start:

* PostgreSQL
* Kafka
* Keycloak
* Elasticsearch
* Fluentd
* Kibana
* Backend services
* Angular frontend

This allows the full platform to run locally.

---

## 10.2 Kubernetes

The platform should later be deployed to Kubernetes.

Kubernetes resources should include:

* Deployments
* Services
* ConfigMaps
* Secrets
* Ingress
* Persistent volumes where required
* Liveness probes
* Readiness probes

---

## 10.3 OpenShift

The platform should also support OpenShift deployment.

OpenShift-specific resources may include:

* Projects/namespaces
* Routes
* ImageStreams, optional
* BuildConfigs, optional
* OpenShift-compatible security settings

The goal is to understand how the same containerized platform can run in an enterprise Kubernetes environment.

---

## 11. CI/CD Strategy

GitHub will be used for source control.

Jenkins will be integrated with GitHub through webhooks.

Recommended flow:

```text
Developer pushes code to GitHub
      |
GitHub webhook triggers Jenkins
      |
Jenkins runs tests
      |
Jenkins builds Java services and Angular frontend
      |
Jenkins builds Docker images
      |
Jenkins pushes images to a container registry
      |
Jenkins deploys to Kubernetes/OpenShift
      |
Jenkins runs smoke tests
```

Recommended registry options:

* GitHub Container Registry
* Docker Hub
* OpenShift internal registry

Recommended starting option:

```text
GitHub Container Registry
```

---

## 12. Repository Structure

Recommended monorepo structure:

```text
cloudops-financial-platform/
│
├── frontend/
│   └── cloudops-ui/
│
├── backend/
│   ├── api-gateway/
│   ├── user-service/
│   ├── operation-service/
│   ├── validation-service/
│   ├── approval-service/
│   ├── notification-service/
│   ├── audit-service/
│   └── incident-service/
│
├── infra/
│   ├── docker-compose/
│   ├── k8s/
│   ├── openshift/
│   ├── fluentd/
│   └── keycloak/
│
├── docs/
│   ├── architecture.md
│   ├── local-development.md
│   ├── kafka-events.md
│   ├── authentication.md
│   ├── observability.md
│   ├── deployment-kubernetes.md
│   ├── deployment-openshift.md
│   └── troubleshooting.md
│
├── Jenkinsfile
└── README.md
```

A monorepo is recommended initially because it is easier to manage and easier to present as a portfolio project.

---

## 13. Suggested Implementation Phases

## Phase 1 — Foundation

Build the basic structure:

* GitHub repository
* Initial README
* Docker Compose base infrastructure
* Keycloak
* PostgreSQL
* Kafka
* API Gateway
* One or two backend services
* Angular frontend skeleton

Goal:

```text
User can log in and access a protected Angular page through Keycloak.
```

---

## Phase 2 — Core Operation Flow

Build:

* Operation Service
* Notification Service
* Kafka event publishing/consuming
* Basic operation creation UI

Goal:

```text
User creates operation -> Operation Service stores it -> Kafka event is published -> Notification is created.
```

---

## Phase 3 — Validation and Approval

Build:

* Validation/Risk Service
* Approval Service
* Approval UI
* Manager approval flow

Goal:

```text
Risky operations require approval before being completed.
```

---

## Phase 4 — Audit and Search

Build:

* Audit Service
* Elasticsearch audit storage
* Audit search UI

Goal:

```text
Every important event is searchable by auditors.
```

---

## Phase 5 — Centralized Logging

Build:

* Structured logs
* Fluentd integration
* Elasticsearch log indices
* Kibana dashboards

Goal:

```text
Logs from all services can be searched centrally in Kibana.
```

---

## Phase 6 — Incidents and Platform Health

Build:

* Incident Service
* Platform health dashboard
* Automatic incident creation for selected failures

Goal:

```text
The platform can track operational failures and expose service health.
```

---

## Phase 7 — Kubernetes and OpenShift

Build:

* Kubernetes manifests
* OpenShift manifests
* Routes/Ingress
* ConfigMaps
* Secrets
* Health checks

Goal:

```text
The platform runs outside local Docker Compose.
```

---

## Phase 8 — Jenkins CI/CD

Build:

* Jenkins pipeline
* GitHub webhook integration
* Docker image build
* Image push to registry
* Automated deployment
* Smoke tests

Goal:

```text
A push or merge in GitHub can trigger build, test, image publishing, and deployment.
```

---

## 14. Future Low-Level Specification Sections

The following sections will be added later as the project becomes more detailed:

* Detailed data model
* Service-by-service responsibilities
* REST API contracts
* Kafka topic definitions
* Kafka message schemas
* Authentication and authorization rules
* Keycloak realm configuration
* Angular routing and UI structure
* Database schema design
* Error handling strategy
* Retry and dead-letter topic strategy
* Logging format
* Correlation ID strategy
* Kubernetes resource definitions
* OpenShift deployment details
* Jenkins pipeline definition
* Testing strategy
* Local development setup
* Production-like deployment setup
* Troubleshooting guide

---

## 15. Initial Architecture Statement

CloudOps Financial Platform is a full-stack, cloud-native, event-driven platform built with Angular and Java Spring Boot microservices. It uses Keycloak for authentication, Kafka for asynchronous service communication, PostgreSQL for transactional business data, Elasticsearch for audit and log search, Fluentd for log collection, and Kibana for observability dashboards. The platform is containerized with Docker, deployable to Kubernetes and OpenShift, and integrated with GitHub and Jenkins for CI/CD.
