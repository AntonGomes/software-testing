## Objectives

- Verify that all functional requirements are met.
- Assess the API’s ability to handle typical and peak loads, ensuring response times remain within acceptable limits.
- Confirm that input validation prevents malicious or malformed HTML from compromising the system or external services, and that the API handles failures gracefully.
- Ensure the test set can adapt whenever new functionality (e.g. new HTML cleaning requirements) is introduced.

## Scope

**In-Scope**:  
- All API endpoints related to retrieving and updating `site-settings` for product pages, including `main` and `tile` scenarios. 
- The LLM integration for CSS selector extraction.  
- Error handling and performance under reasonable loads.
**Out-of-Scope**:  
- Front-end tests (the report and plan focus on back-end/API only).  
- Any external data sources not used in this project).

## Environment

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

## Test Data

Sufficient test data must be gathered to ensure rigorous testing. The test data will include a range of HTML files with the following features:
- `script` tags.  
- Is abnormally long i.e. would breach the context window of the LLM.  
- Product(s) with price.  
- `head` tags which contain the retialer name of the site owner.  
- Represents a `"main"` product page.  
- Represents a `"tile"` product page.

For each HTML file, the smallest `container` for the product(s) should be identified, as well as the smallest container for the product `image`, `description`, and `price`.

## Strategy

#### Unit Tests
- Verify parsing logic returns valid CSS selectors for each product attribute (FR 1).
- Validate schema errors for invalid site-settings (FR 2).
- Confirm partial updates do not overwrite existing selectors (FR 3).
- Confirm path parameters accept only valid values and instantiate correct classes/prompts (FR 4–7, FR 9).
- Ensure CleanHTML handles scripts, large HTML, price elements, and head tags properly (FR 10.1–10.4).
- Simulate LLM failures/timeouts and large or malformed HTML to check graceful error handling (RR 1.1, RR 1.2).
- Verify partial site-settings updates if some extraction steps fail (RR 3).

#### Integration Tests

- Combine CleanHTML output with LLM prompts to confirm correct data flow (FR 1, FR 9).
- Check that updated site-settings data persists across multiple endpoints (FR 2, FR 3).
- Validate concurrency on a small scale, ensuring correct handling of simultaneous requests (PR 2).

#### System Tests

- End-to-end scenarios where HTML is submitted, the LLM is invoked, and site-settings are updated (FR 1–10).
- Check overall performance for short-to-moderate HTML (PR 1).
- Confirm security constraints: input validation for malicious HTML, environment variable usage for OpenAI API keys, and no secret data leakage (SR 1, SR 2).

#### Performance and Stress Tests

- **Load Testing (PR 1, PR 2)**
	- Measure average/peak response times under normal and concurrent load.
	- Identify bottlenecks in `CleanHTML` or LLM calls.
- **Stress Testing (RR 1.2)**
	- Push beyond typical concurrency or HTML size to observe system failure modes and error responses.

## Risk Analysis

- If the OpenAI API changes, breaks, or experiences downtime, tests may fail or become irrelevant.  
	  *Mitigation*: Use mock services to replicate responses and reduce dependency on real endpoints.
- Highly variable or large HTML inputs might exceed time or token limits.  
	  *Mitigation*: Include edge-case HTML samples.
- If new attack vectors emerge (e.g., advanced script injections), current sanitisation might be insufficient.  
	  *Mitigation*: Periodic security reviews and updated sanitisation checks within the test suite.

## TDD

Below describes an example case where TDD was used to implement a new requirement of the software. 

#### Example: Updating CleanHTML to Only Pass `<title>` Content for Retailer Name

Suppose you have a new requirement: when calling the `find-retailer-name` endpoint, only the contents of the `<title>` tag (rather than the entire `<head>`) should be passed to the LLM.

1. **Define the Requirement and Write the Test:**
	- Create a test that checks how the CleanHTML class processes an HTML file containing various `<head>` elements, ensuring only `<title>` content is returned for the find-retailer-name functionality.
	- Expect the test to fail at first because the code does not yet isolate `<title>` content.

2. **Implement the Feature:**
	- In CleanHTML, add or modify a method (e.g., `clean_find_retailer_name_html`) to parse out just the `<title>` tag content and discard other `head` elements.

3. **Run and Pass the Test:**
	- Execute the test. If the new code meets the requirement, the test passes.

4. **Refactor if Needed:**
	- If the code can be made more efficient or readable, refactor it.
	- Re-run the test suite to confirm that all functionality, including the new `<title>` extraction, still works.

