## 1. Functional Requirements

**FR 1.** Given the HTML for a webpage from an e-commerce site, the API must accurately identify and return CSS selectors for specific product information (`images`, `description`, `price`, `link`), or a valid *outer container* which should contain all the elements hosting the aforementioned product information. Note that the specific selector or corresponding element is not important; it is only the content of the element that must be accurate. For example:

```html
<div id="title">
    <h1>This is a product title</h1>
</div>
```

Here both `#title` and `h1` are valid selectors. Note also that we only care about the selector in the context of the element identified by the container selector. An accuracy rate of 95% is acceptable for this requirement to be met. 

**FR 2.** The site-settings payload parameter must fit the `site_settings_model` Pydantic data model. A type-checking error here should result in an error response.

**FR 3.** The API must correctly update the site-settings data model based on new CSS selectors while preserving any previously stored selectors.

**FR 4.** The gpt-model path parameter must only accept the values `“gpt-4o”` or `“gpt-4o-mini”`. Any other value must result in an error response.

**FR 5.** The LLM class should be instantiated with the correct model according to `gpt-model`.

**FR 6.** The `product-type` path parameter must only accept the values `“main”` or `“tile”`. Any other value must result in an error response.

**FR 7.** The correct `ProductFinder` subclass should be instantiated according to `product-type`.

**FR 8.** The API should be able to extract accurate selectors for both `“main”` and `“tile"` pages.

**FR 9.** The correct prompt must be used according to the endpoint being called and the given `product-type`.

**FR 10.** The raw HTML should be cleaned according to the product information being extracted. The class `CleanHTML` is responsible for the following requirements:
- **FR 10.1.** Scripts and irrelevant tags should always be removed.
- **FR 10.2** Truncation of HTML if content surpasses LLM context window.
- **FR 10.3.** Correct identification of HTML elements containing product prices for the `find-price-container` endpoint.
- **FR 10.4.** Identification and cleaning of `<head>` elements for the `find-retailer-name` endpoint.

## 2. Performance Requirement

**PR 1.** The API must process and return responses within a reasonable time, ideally under 10 seconds. This timing includes:
- HTML cleaning (via CleanHTML).
- Constructing and sending a request to the OpenAI API.
- Parsing the OpenAI API response into a suitable data model.
	
**PR 2.** The API should be tested under moderate levels of concurrency (e.g., multiple simultaneous requests) without significant performance degradation.

## 3. Security Requirements

**SR 1.** Input validation must ensure that malicious or malformed HTML does not compromise the API or external services. This includes cleaning the HTML to remove scripts and harmful tags.

**SR 2.** Sensitive data (e.g., OpenAI API keys) must be stored securely using environment variables and must not be exposed in client-facing responses or logs.

**SR 3.** Rate limiting or usage monitoring should be employed to prevent abuse of the LLM endpoints.

## 4. Robustness Requirements

**RR 1.** The API must handle failures gracefully, including:
- **RR 1.1.** LLM unavailability, timeouts, or errors (e.g., return a clear error message or a fallback strategy).
- **RR 1.2** Invalid, incomplete, or excessively large HTML input (e.g., apply additional truncation or reject requests).

**RR 2.** If the LLM fails to identify a CSS selector, the API should return a clear and actionable error response or log an appropriate warning.

**RR 3.**  If the LLM fails to identify a CSS selector, the site-setting should not be updated. 
