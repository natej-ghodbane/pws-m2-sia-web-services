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