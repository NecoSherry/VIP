Reviewers Guidelines
====

## Responsibilities
Upon the arrival of each new VIP, an reviewer undertakes the following steps:
+ Read a new VIP to check if it makes technical sense
+ Check the language and format
+ Assign an VIP number
+ Form a working group to assist the authors
+ Approve the pull request if there is no issues when corresponding VIPs change status
+ Merge the corresponding pull request
+ Convene meetings as necessary

## Rules of assigning VIP number
The number consists of three digits: the first two digits represent the year, and the third digit (or the last two digits) indicates the index of a VIP within that year.

For example, let's take the number 220:

The first two digits, 22, represent the year, which in this case would be 2022.
The third digit, 0, indicates that the VIP associated with this number is the first VIP of the year 2022.
So, the assign number 220 would correspond to the first VIP of the year 2022.

Here's another example: 
Let's say we have the number 2312:
The first two digits, 23, represent the year, which would be 2023.
The last two digits, 12, indicate that this number represents the thirteenrd VIP of the year 2023.
Therefore, the assign number 2312 would correspond to the thirteenrd VIP of the year 2023.

That's how the assign number system works, where the first two digits represent the year, and the third (or last two) digits represent the position of the VIP within that year.

## Review checklist
### New proposal
For each new VIP that comes in, it need to meet the requirements:
+ The relevance - any new proposal submitted is related to Vechain blockchain.
+ The title describing the content is accurate,
+ The idea making sense technically, even if they don’t seem likely to get to final status.
+ The language (spelling, grammar, sentence structure, etc.), markup (GitHub flavored Markdown), code style and the proposal is well formatted.

### Status Changed
once the author create a pull request to change the status to next stage, to check a new proposal for it’s readiness to ensure the proposal meets the requirements of the target status.

#### `Idea` to `Draft`
The proposal need to include:
+ POC
+ Feasibility validation: including Technical Feasibility, Feasibility Study and implementation plan.
+ a feasibility assessment report based on the validation results and research findings.
+ Consider incorporating relevant discussions from the Discourse platform to enhance the proposal. 

#### `Draft` to `Acccepted`
The proposal need to meet the requirements:
+ evaluate its technical viability and compliance with VeChain's standards and capabilitie by reviewing the code and the poc process
+ The community, reviewers, and the author reach a consensus that there is no issue on this proposal.

#### `Accepted` to `Final`
The proposal need to meet the requirements:
+ Has been tested to ensure that it functions as intended and does not introduce any security vulnerabilities. 
+ The implementations is deployed on testnet and mainnet. Has no issues about the implementations.
+ Implementation details have been updated on GitHub.
+ The VeChain Foundation coordinates the necessary upgrades and ensures a smooth transition without disrupting the ecosystem's operation.

