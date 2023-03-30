# Notes on PACT contract testing

Integrations tests are about functionality while contract tests are about the interface.

> These tests are not component tests. They do not test the behaviour of the service deeply but that the inputs and outputs of service calls contain required attributes and that response latency and throughput are within acceptable limits.
> Ideally, the contract test suites written by each consuming team are packaged and runnable in the build pipelines for the producing services. In this way, the maintainers of the producing service know the impact of their changes on their consumers.

![Screenshot 2023-03-02 at 9 30 17 AM](https://user-images.githubusercontent.com/17195702/222373759-754e38a3-bddd-4480-ac39-56aa5991ad89.png)

## Pact

- Pact is a contract testing tool that further improves the contract testing workflow by allowing users to implement consumer-driven contracts.
- Contract testing is an alternative approach to traditional integration testing.

Pact on its own allows you to create and verify contracts. Pact + Pact Broker allows you to integrate contract testing into your CI/CD pipeline to allow you to release code faster.

### How it works

- During the consumer tests, each request made to a Pact mock provider is recorded into the contract file (the Pact file), along with its expected response. The Pact mock provider can be a substituted with an adapter, for instance if WireMocks or MSW.
- A Pact simulated consumer then replays each request against the *real provider*, and compares the actual and expected responses. 
- If they match, we have verified that the simulated applications behave the same way as the real applications. 
- When contract testing is in place we use Pact Broker to: share contract across teams, manage contracts across branches, orchestrate builds.

https://pactflow.io/how-pact-works/#slide-1

## Pact Broker

- The Pact Broker is an application for sharing consumer driven contracts and verification results.
- It is open source, there is a plug-and-play version payed called PactFlow.
- Pact Broker can be used via Docker https://hub.docker.com/r/pactfoundation/pact-broker
- Contract testing gives you tests that are quicker to execute. One down side of the approach is that the important information that would be available all in one place at the end of an integration test suite execution. The Pact Broker is a tool that brings all this information back together again.

Pact Broker is useful for:

- Integration with automate contract testing in CI/CD and allow deploying service indipendently
- Solve the problme how to share contracts and verify the results between consumer and providers
- Versions management
- Allows you to ensure backwards compatibility between multiple consumer and provider versions 
- Alows you to visualise and document the relationships between your services, digrams and charts

Main features:

- RESTful API for publishing and retrieving pact files
- API browser to navigate the API
- Autogenerated docs
- Autogenerated diagrams for microservice networks
- Provides a "matrix" of compatible consumer and provider versions
- Webhooks to trigger actions when pacts change or verification results are published eg. run provider build, notify a Slack channel.
- Vew diffs between Pact versions
- CLI encorporating Pact workflow

How it is working:

Step 1

- The consumer project runs its tests using the Pact library to provide a mock service (or use an adapter like for WireMocks or MSW)
- While the tests run, the mock service writes the requests and the expected responses to a JSON "pact" file - this is the *consumer contract*.
- The generated pact file is then published to the Pact Broker (the easiest way to do this is to publish the pact file using the Pact Broker Client CLI)
- When a pact file is published, a webhook in the Pact Broker kicks off a build of the provider project if the pact content has changed since the previous version and requires verification.

Step 2

- The provider has a verification task that is configured to retrieve the relevant pacts between itself and its consumer.
- The provider build runs the pact verification task, which retrieves the pact(s) from the Pact Broker, *replays each request against the provider*, and checks that the responses match the expected responses.
- If the pact verification fails, the build fails. 
- The results of the verification are published back to the Pact Broker by the pact verification tool, so the consumer team will know if the code they have written will work in real life.
- The Provider CI determines if the provider is compatible with its consumers in a particular environment.
- If the pact has been verified successfully, the deployment can proceed.
- When the provider is deployed they record the deployment with Pact.

Step 3. Back to the Consumer CI build

- The Consumer CI determines if the pact has been verified.
- If the pact has been verified successfully, the deployment can proceed.
- When the consumer is deployed they record the deployment with Pact .

https://docs.pact.io/pact_broker

https://martinfowler.com/articles/microservice-testing


## Bi-Directional Contract Testing (BDCT)

Bi-Directional Contract Testing is a type of static contract testing where two contracts - one representing the consumer expectations, and another representing the provider capability - are compared to ensure they are compatible.

> The comparisons is based on their schema.

BDCT is a feature exclusive to PactFlow and is not available in the Pact OSS project (it is a paid solution).

- Contract first provider driven workflows.
- Using OpenAPI.
- Integration with MSQ, Cypress, Wiremock BYO (Bring Your Own mocks).
- Support contract verification Postman, Dredd.
- For white/black box testing.
- Wide audience contract testers: dev, qas, SDET (Software Development Engineer in Test).

https://docs.pactflow.io/docs/bi-directional-contract-testing/?&utm_source=bdct-lander&utm_campaign=bdct-go-live&utm_content=how-it-works

Pact is a record and replay style contract using specification by example to prove correctness.
*Bi-Directional Contract Testing instead using a schema comparison approach*.

Packflow provides continuous api compliance by comparison the consumer contract with the provider contract such as an API contract.
It allows to generate of contracts from existing mocks (Generated by tooling like WireMocks, MSW or others). 
Developers can use their plug-and-play adapter.
This is good because you can leverage existing tools.

Pros:
- Low barrier to entry.
- Scale faster, reuse existing tests.
- Upscale re-using current processed and tolling.

## How it works

### Consumer side

- Consumer tests behaviour against a mock (write a test that checks our consumer behavior against a mock), we could use WireMocks or MSW in this part
- From the mock used to test the consumer, we use it to generate to contract (pact file, aka the consumer contract).
- Test consumer and provider independently.

### Provider side

- We start with a provider contract (for instance OpenAPI spec, generated by hand or from source code).
- We do provider testing, make sure the provider code base is compatible with that OpenAPI spec (verify provider contract).
- The end result we have 2 contracts, one what the consumer needs and another what provider can do.
- Both contracts are uploaded on Pactflow, the consumer and provider have separated pipelines.
- After a contract comparison is made, to make sure their consumer is a valid subset of the provider contract (schema comparison).

## Steps

## Consumer

- The consumer tests its behavior against a mock such Wiremock, MSW or Pact (Pact or an adapter will generate the pact file).
- Consumer contract is produced in the form of pact file, there only the interaction with the consumer is captured.
- Consumer contract is uploaded to Pactflow.
- We check the compatibility with the provider.

## Provider

- The provider starts with its own specification (OpenAPI), this is the provider contract (can be generated by hand or by source code), if OpenAPI is generated by code we can skip the next point.
- The provider contract is tested against the provider using a funcional API testing tool such RestAssured, Dredd, or Postman.
- Provider contract i uploaded to Pactflow.
- We check the compatibility with the consumer.

## Notes:

With BDCT, the key difference is that a Provider uploads its own provider contract advertising its full capacity which is *statically compared* to the expectations in the consumer contract - *the consumer contract is never replayed against the provider code base*. This creates a much simpler and decoupled workflow. See the trade-offs for more.

## Remember:

- Contract tests focus on the messages that flow between a consumer and provider (schemas).
- Functional tests also ensure that the correct side effects have occurred (for instance a new entry is created in db).
- A consumer contract is a collection of interactions that describe how the Consumer expects the Provider to behave. Each Consumer will have its own unique consumer contract for each of its Providers.
- A provider contract specifies the capability of the Provider, this could take the form of an OpenAPI document, but may be other formats such as a GraphQL schema, a SOAP XSD, a protobuf definition and so on.

https://pactflow.io/difference-between-consumer-driven-contract-testing-and-bi-directional-contract-testing/

https://pactflow.io/how-pact-works/#slide-1

![Screenshot 2023-03-02 at 10 00 45 AM](https://user-images.githubusercontent.com/17195702/222381222-fb43bc17-a71f-43f9-b263-ae35c4459a7d.png)

## Videos

https://www.youtube.com/watch?v=j0xe2dfAI-I

https://www.youtube.com/watch?v=VmluCw6a6IE

https://www.youtube.com/watch?v=V-OV6lRwhYA


![Screenshot 2023-03-30 at 12 38 24 PM](https://user-images.githubusercontent.com/17195702/228811261-d324811e-57d8-4078-8302-72d6d040fae8.png)
