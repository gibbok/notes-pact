# Notes on PACT contract testing


## Bi-Directional Contract Testing (BDCT)

It is a new form of contract testing based on schema testing.

- Contract first provider drivern workflows
- Using OpenAPI
- Integration with MSQ, CYpress, Wiremock
- Suppot contract verification Postman, Dredd
- For white/black box
- Wide audiance contract tester dev, qas, SDET (Software Development Engineer in Test)

https://docs.pactflow.io/docs/bi-directional-contract-testing/?&utm_source=bdct-lander&utm_campaign=bdct-go-live&utm_content=how-it-works


Pact is a record and replay style contract using specification by example to prove correctness.
Bi-Directional Contract Testing instead using a schema comparison approch.

Packflow provide continuose api compliance by comparisni the consumer contract with the provider contract such an API contract.
It allow to generate contract from existing mocks (such as wiremock). 
