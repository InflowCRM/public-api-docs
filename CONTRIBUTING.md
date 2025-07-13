# Contributing

Currently, only the repository owner manages this documentation.

If you want to report a bug or request a feature, please use the GitHub Issues tab and select the appropriate template (Bug Report or Feature Request).

Thank you!

## OpenAPI Specification Linting

To maintain the quality and consistency of our API documentation, we use [Spectral](https://github.com/stoplightio/spectral) to lint the `openapi.yaml` file. This is enforced via a GitHub Action that runs on every pull request.

### Local Validation

Before submitting a pull request, you can validate your changes locally by running:

```bash
npm install -g @stoplight/spectral-cli
spectral lint docs/rest-api/openapi.yaml
```