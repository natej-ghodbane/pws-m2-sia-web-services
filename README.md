# Shop — Web Services M2 SIA

Lab 1: the Inventory service, behind two doors.

## Run it

    python -m uvicorn services.inventory.main:app --reload --port 8000

## Questions

### 1. DELETE /products/42

The request returned:

    HTTP/1.1 405 Method Not Allowed

The `allow` header said:

    allow: GET

This is a 405 rather than a 404 because the resource `/products/42` exists,
but the HTTP method DELETE is not allowed for this route.
Only GET is allowed.

### 2. SOAP failure

When requesting product 99 through SOAP, the HTTP status line still said:

    HTTP/1.1 200 OK

The failure was only represented inside the SOAP XML body using an `env:Fault`.

This can mislead infrastructure such as a monitoring system, a proxy, a cache,
or a load balancer because they only see the HTTP status code and may consider
the request successful even though the SOAP operation actually failed.

### 3. Which door is easier for a colleague to use?

The HTTP door is easier to use.

For HTTP, a colleague mainly needs the URL and the HTTP method:

    GET /products/42

They can also inspect the automatically generated FastAPI documentation at `/docs`.

For SOAP, a colleague needs more information:
- the endpoint URL `/ProductService`
- the POST method
- the XML envelope structure
- the SOAP namespace
- the shop namespace
- the operation name `GetProduct`
- the expected XML element names

Therefore the HTTP interface is easier to discover and use without additional explanation.

## Lab 2 — REST contract, SQLite, Orders service, Docker Compose

## Run it

    docker compose up --build -d

Inventory: http://127.0.0.1:8000/docs  
Orders: http://127.0.0.1:8001/docs

## Lab 2 questions

### 1. PUT vs POST order and idempotency

Sending the same PUT request twice leaves the product in the same final state.

The second PUT does not create another product and does not introduce an additional change. This is why PUT is idempotent.

Sending the same order twice with POST is different. Each POST creates a new order and reserves the stock again.

Therefore, after a timeout, a client may safely resend the PUT request without asking first. It should not automatically resend the POST order because it could create a duplicate order and reserve the stock twice.

### 2. Why use POST /products/{id}/reservations?

Orders should not first read the stock with GET and then update it with PATCH.

Imagine that only one unit is left. Two orders arrive at the same time:

1. Order A reads stock = 1.
2. Order B also reads stock = 1.
3. Order A decides that the product is available.
4. Order B also decides that the product is available.
5. Both try to reduce the stock.

Both orders could therefore believe that they obtained the last unit.

The reservation endpoint avoids this problem because Inventory checks the available stock and subtracts the requested quantity in a single database operation.

### 3. Why does Orders return 503 when Inventory is stopped?

When Inventory is stopped, the HTTP request from Orders to Inventory fails.

The `try/except` catches this communication error and Orders returns:

    HTTP/1.1 503 Service Unavailable

Without the `try/except`, the exception would not be handled and the client would receive a `500 Internal Server Error`.

A 500 would be misleading because the problem is not necessarily a bug in Orders. The problem is that one of its dependencies, Inventory, is unavailable.

Therefore, 503 is more appropriate because it tells the client that the service is temporarily unable to complete the request.

### 4. Why does Docker Compose use http://inventory:8000?

When the services run locally, Orders can reach Inventory using:

    http://127.0.0.1:8000

However, with Docker Compose, Orders and Inventory run inside separate containers.

Inside the Orders container, `127.0.0.1` refers to the Orders container itself, not to the Inventory container.

Docker Compose creates a network for the services and uses the service names as hostnames.

Therefore Orders reaches Inventory using:

    http://inventory:8000