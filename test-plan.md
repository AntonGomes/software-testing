# Introduction

# Objectives
- Verify that all functional requirements are met.
- Assess the API’s ability to handle typical and peak loads, ensuring response times remain within acceptable limits.
- Confirm that input validation prevents malicious or malformed HTML from compromising the system or external services, and that the API handles failures gracefully.
- Ensure the test set can adapt whenever new functionality (e.g. new HTML cleaning requirements) is introduced.

# Scope
- **In-Scope**:  
  - All API endpoints related to retrieving and updating `site-settings` for product pages, including `main` and `tile` scenarios.  
  - The LLM integration for CSS selector extraction.  
  - Error handling and performance under reasonable loads.
- **Out-of-Scope**:  
  - Front-end tests (the report and plan focus on back-end/API only).  
  - Any external data sources not used in this project).

# Environment
- **OS**: MacOS Sonoma 14.5
- **Key Software/Frameworks**:
  - Python 3.12.6
  - FastAPI 0.104.1
  - Pydantic 2.7.4
  - pytest 8.3.4
  - requests 2.31.0
  - openai 1.3.4
- **External Services**:
  - OpenAI API (mocked where needed to simulate various responses).

# Test Data
Sufficient test data must be gathered to ensure rigorous testing. The test data will include a range of HTML files with the following features:

- `script` tags.  
- Is abnormally long i.e. would breach the context window of the LLM.  
- Product(s) with price.  
- `head` tags which contain the retialer name of the site owner.  
- Represents a `"main"` product page.  
- Represents a `"tile"` product page.

For each HTML file, the smallest `container` for the product(s) should be identified, as well as the smallest container for the product `image`, `description`, and `price`.

# Strategy

## Unit Tests
Each unit test will correspond to a functional requirement, thereby ensuring all are met.

- Test each functionality of the `CleanHTML` class (1.5):
  - Detection/removal of scripts and irrelevant tags.
  - Truncation of HTML if content surpases LLM context window.
  - Correct identification of HTML elements containing product prices.
  - Identification and cleaning of `<head>` elements for `find-retailer-name` endpoint.
- Ensure the correct GPT model and prompt are used based on `product_type` and attribute request (1.4).
- Check that exceptions are raised when the LLM is unavailable or HTML is invalid (4.1).

## Integration Tests
- Confirm seamless data flow between HTML cleaning and LLM modules, verifying that partial or malformed inputs result in appropriate error messages (1.5, 4.1.1).
- Validate that updated selectors integrate correctly with existing `site-settings` data (1.2).

## System Tests
- Verify the system end-to-end, from receiving raw HTML to returning final CSS selectors for multiple product types (1.1, 1.3).
- Test how the system responds to multiple simultaneous calls, ensuring performance remains acceptable (2.1).

## Performance and Stress Testing
- Incrementally increase each of the following loads to identify points of failure or severe slowdown:
  - Concurrent requests (2.1).
  - Request volume.
  - HTML size.

# Risk Analysis
- If the OpenAI API changes, breaks, or experiences downtime, tests may fail or become irrelevant.  
  *Mitigation*: Use mock services to replicate responses and reduce dependency on real endpoints.
- Highly variable or large HTML inputs might exceed time or token limits.  
  *Mitigation*: Include edge-case HTML samples.
- If new attack vectors emerge (e.g., advanced script injections), current sanitisation might be insufficient.  
  *Mitigation*: Periodic security reviews and updated sanitisation checks within the test suite.
