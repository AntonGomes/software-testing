## Objectives

- Verify that all functional requirements are met.
- Assess the API’s ability to handle typical and peak loads, ensuring response times remain within acceptable limits.
- Ensure complete input validation. This includes cleaning malformed HTML.
- Ensure the test set can adapt whenever new functionality (e.g. new HTML cleaning requirements) is introduced.
- Ensure errors are handled gracefully.

## Scope

**In-Scope**:  
- All API endpoints related to retrieving and updating `site-settings` for product pages, including `main` and `tile` scenarios. 
- The LLM integration for CSS selector extraction.  
- Error handling and performance under reasonable loads.
**Out-of-Scope**:  
- Front-end tests (the report and plan focus on back-end/API only).  

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

Test data refers to the HTML files used in testing the HTML parser and to assess the accuracy of the LLM in retrieving CSS selectors. These two scenarios have different base and edge cases outlined below:

**HTML Parsing Data**
- `BASE_TILE_HTML`: A standard product tile page taken from asos.com. `<head>` tags contain the retailer name. Contains "malicious" scripts, Contains exactly 20 price elements. 
- `BASE_MAIN_HTML`: A standard main product page taken from asos.com. `<head>` tags contain the retailer name. Contains "malicious" scripts. Contains exactly 20 price elements. 
- `NO_HEAD_HTML`: Does not contain `<head>` tags. 
- `EMPTY_HEAD_HTML`: `<head>` element is empty.
- `NO_PRICE_HTML`: Contains no price elements.
- `LONG_HTML`: A html file which is larger than the context window of the LLM. 

**CSS Selector Identification Data**
There should be 10 distinct data points of the form:
- `llm_test_data/input/X_TILE_HTML`: A standard tile product page from an e-commerce site. 
- `llm_test_data/input/X_MAIN_HTML`: A standard main product page from an e-commerce site `X`. 
- `llm_test_data/output/X_SITE_SETTINGS`: The complete site settings for site `X` with the CSS selectors matching the largest possible container in each case. 

## Strategy

#### Unit Tests
- Verify full input validation and check errors are handled gracefully (FR 2, FR 4, FR 6). 
- Verify site-settings are correctly updated when a CSS selector is identified (use mock LLM API) (FR 3). 
- Verify the correct LLM is instantiated according to `gpt-model`(FR 5).
- Verify correct classes are instantiated according to `product-type`(FR 7).
- Verify correct prompts are used according to endpoint and `product-type` (FR 9).
- Ensure CleanHTML handles scripts, large HTML, price elements, and head tags properly (FR 10.1–10.4, SR 1).
- Simulate LLM failures/timeouts to check graceful error handling using a mock (RR 1.1).
- Check for graceful error handling if the LLM is passes invalid HTML (RR 1.2).
- Check excessively large HTML files are properly truncated (RR 1.2). 
- Verify graceful error handling if the LLM is unable to detect a CSS selector (RR 2). 
- Verify site-settings remain unchanged if a CSS-selector fails to be identified.

#### Integration Tests

- Combine CleanHTML output with LLM prompts to confirm correct data flow (FR 1, FR 9).
- Check that updated site-settings data persists across multiple endpoints (FR 2, FR 3).

#### System Tests

- Measure accuracy rate of the full system when tested on each example html file. Present accuracy results by product type and product information type. (e.g. Main-images: 95%, Tile-price: 85%)
	A CSS-selector is considered accurate if the container it identifies is equal to or contained within the container identified by the given site-settings "solution".
#### Performance and Stress Tests

- Test API response time for each endpoint and each product type. Use mock LLM API, adding the given standard response time of the actual LLM API to auto-site-setting's response time (PR 1). 
- Verify API response time remains within 10 seconds under concurrency levels 5, 10, 15. Use mock LLM API (PR 2). 
- Confirm rate limiting is in place (SR 3).

## Risk Analysis

- If the OpenAI API changes, breaks, or experiences downtime, tests may fail or become irrelevant.  
	  *Mitigation*: Use mock services to replicate responses and reduce dependency on real endpoints.
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

## Success Criteria

- All known edge cases (e.g., missing `<head>` tags, empty `<head>`, large HTML files, and HTML with no price elements) are successfully handled to ensure parser robustness.
- Complete path parameter validation for API endpoints, ensuring invalid or malformed parameters are handled gracefully.
- Type and data validation for `site-settings` objects, ensuring data integrity.
- Achieve 100% code coverage across all unit, integration, and system tests to ensure all critical paths are tested.
- The API must meet an average response time of 10 seconds or less under typical loads, including simulated OpenAI API calls.
- Achieve 95% accuracy in identifying CSS selectors, ensuring that the largest relevant containers are consistently selected.
- Graceful handling of all simulated errors, including OpenAI API failures, timeouts, and invalid HTML inputs, without system crashes.