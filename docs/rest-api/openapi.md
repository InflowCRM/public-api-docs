# OpenAPI Specification

The InflowCRM Public API provides an OpenAPI 3.0 specification that describes all REST endpoints, request/response schemas, and authentication requirements.

## What It Covers

- All REST API endpoints (Records, Users, Files, Webhooks, Utility)
- Request and response schemas with examples
- Authentication via `x-api-key` header
- Error response formats
- Advanced filtering schemas

> **Note:** The GraphQL API is not covered by this spec — see the [GraphQL documentation](docs/graphql/schema.md) instead.

## Download

- [openapi.yaml](https://github.com/InflowCRM/public-api-docs/blob/main/docs/rest-api/openapi.yaml) (GitHub)
- [openapi.yaml (raw)](https://raw.githubusercontent.com/InflowCRM/public-api-docs/main/docs/rest-api/openapi.yaml)

## How to Use

### Import into Postman
1. Open Postman and click **Import**
2. Select **Link** and paste the raw YAML URL above
3. Postman will generate a collection with all endpoints pre-configured
4. Set the `x-api-key` variable in your environment

### Import into Insomnia
1. Open Insomnia and go to **Application > Import**
2. Select **From URL** and paste the raw YAML URL
3. Configure the `x-api-key` header in the base environment

### Interactive Viewer (Swagger UI)

You can preview the spec interactively using Swagger UI:

```
https://petstore.swagger.io/?url=https://raw.githubusercontent.com/InflowCRM/public-api-docs/main/docs/rest-api/openapi.yaml
```

### Redoc Viewer

For a clean read-only reference, use Redoc:

```html
<!DOCTYPE html>
<html>
<head><title>InflowCRM API Reference</title></head>
<body>
  <redoc spec-url="https://raw.githubusercontent.com/InflowCRM/public-api-docs/main/docs/rest-api/openapi.yaml"></redoc>
  <script src="https://cdn.redoc.ly/redoc/latest/bundles/redoc.standalone.js"></script>
</body>
</html>
```