VeChain Improvement Proposals
====
## Overview

A VIP, which stands for VeChain Improvement Proposal, is a design document that serves the purpose of providing information or describing a new feature within the VeChain ecosystem. It outlines new standards for various aspects such as the core protocol, client APIs, digital assets, smart contract development and anything that shapes certain dApps built on the VeChainThor blockchain. A key requirement of a VIP is to offer a clear and concise technical specification of the proposed feature.

Additionally, the author of a VIP carries the responsibility of fostering consensus within the community and documenting any dissenting opinions that may arise during the proposal's evaluation process. This ensures transparency and inclusivity in the decision-making process.

We provide [the status page](./the-status-page.md) listing and tracking VIPs.


## Categories:
Here we define the following VIP categories:
+ **Core**: VIPs that focus on describing improvements on the Thor protocol (e.g., the consensus algorithm, transaction model, account model, runtime and etc.)  which may result in a consensus fork. Typically, they require broad agreement and coordination among the VeChain community.
+ **Interface**: VIPs that concentrate on improving client API specifications and standards. They aim to improve interoperability and consistency in client implementations.
+ **Application**: VIPs that establish standards and conventions at the application level. It may include but not limited to standards related to the digital asset type, name registry, smart contract library, SDK for developing dApps, and etc. They provide guidelines for developing and deploying applications within the VeChain ecosystem.
+ **Information**: These VIPs address design issues or offer general guidelines and information to the community. Although they do not propose new features, they serve to share valuable insights and knowledge within the VeChain ecosystem.

## Process 
![VIP](https://github.com/NecoSherry/VIPs/assets/5069216/e275d011-94dc-49e0-abfb-ba87ff43480d)

**Idea**: At this stage, applicants will need to prepare a document that describes their ideas and create a PR on the repo. Meanwhile, they will also need to post their ideas on the dedicated builders forum to further engage with the dev community as well as reviewers for discussion. It is to ensure its originality and significance to the VeChain ecosystem, and avoid wasting time on a potentially rejected proposal. 

**Draft**: Once an idea attracts sufficient support from the reviewers and dev community, the author will be asked to format the document to meet the VIP format requirements, and make necessary changes according to the related discussions. A VIP number will be given to the new VIP and the PR merged to the repo. At this stage, a working group made of relevant reviewers will be formed and then start the formal review process for the VIP. 

**Accepted**: There has been a consensus reached among all the participating reviewers that the VIP has no technical issues and is ready to be implemented.

**Final**: A VIP enters the stage where it has been implemented within the VeChain ecosystem. A VIP in this state is considered complete and closed.

**Stagnant**: Any VIP in `Draft` or `Accepted`  inactive for a period more than 6 months will become Stagnant. 

**Rejected**: A VIP will be marked `Rejected` if reviewers cannot reach a consensus on its technical contents.

**Withdrawn**: A VIP will be marked `Withdrawn` if the author has formally withdrawn it.
**Superseded**: A VIP will be marked `Superseded` if it has been replaced by a newer VIP.

### 1. Submitting a VIP
To submit a VIP, the author should follow these below steps:
1. Write the VIP using the [VIP template](./vip-template.md).
2. Submit a pull request with the VIP to the [vechain/VIPs](https://github.com/vechain/vips) repository.
3. Create a new topic on [the Vechain Builders forum](https://vechain.discourse.group) and share the PR link along with the VIP description to initiate community discussion and gather feedback.
4. Update the VIP based on the community's feedback and suggestions.
5. @sherry under the pull request to request an initial review once the VIP is ready.
6. The proposal will be reviewed by VIP reviewers who will provide feedback and propose changes to improve the proposal.
7. To have the VIP merged, you need to receive more than **3 approvals** from the reviewers on the pull request. All the reviewers who approved the VIP will form a working group and have a private channel for discussion on Discord.
8. The reviewers will assign a number to the VIP and merge the pull request, moving the VIP to the `Draft` stage.

### 2. Refine technical details
For a VIP in `Draft` status:
1. The author of the VIP needs to work on refining the technical details of the proposal until it reaches a stage where it can be practically implemented.
2. The author creates a pull request to update the VIP in the vechain/VIPs repository.
3. The reviewers within the working group will then thoroughly review the technical aspects of the VIP to ensure it could be implemented with the VeChain protocol.
4. For the VIP to move to the `Accepted` stage, it requires unanimous approval from all the reviewers in the working group. This means that all reviewers must agree and have no issues or objections to the proposed VIP.
5. The author should request all the reviewers in the working group to approve and merge the pull request if they are satisfied with the VIP's technical details.


### 3. VIP Implementation
VIPs that on `Accepted` will be implemented in a future release of the VeChain protocol. 

For VIPs in the **Core** category or requiring changes to the client:
1. The core dev team will carry out the actual implementation and testing.
2. The core dev team may seek assistance from the author(s) during implementation.
3. Once successfully implemented, the VIP will be changed to the `Final` status.

For VIPs in other categories:
1. The author of the VIP will be responsible for carrying out testing and implementation.
2. The author can request assistance from the working group during this process.
3. After completing the implementation, the author should submit a pull request to update the VIP with all the necessary details.
4. The VIP will need approval on GitHub from all reviewers within the working group to progress to the `Final` status.

## VIP Reviewers

The current VIP reviewers consist of the following members:

From Core dev teams:
+ Tony Li (@libotony)
+ Xiqing Chu (@laalaguer)
+ Mog Lu (@mongelly)
+ Asbert Ma (@AsbertMa)
+ Peter zhou (@zzGHzz)
+ Bin qian (@qianbin)

From well-known community projects:
+ Fabian (VeChainStats) 
+ Mario (vechain.energy) 
+ MiRei (VeBlocks)
+ NeroMVA (madvape) 

From Vechain EU team:
+ Neil
+ Vineet
+ Ben Moran
+ Daithi

Learn more about [the reviewers' guidelines](./reviewers-guidelines.md).


## Community
+ **GitHub**: https://github.com/vechain/vips 
+ **Forum**: https://vechain.discourse.group 
+ **Discord**: Visit https://discord.com/invite/vechain, join the Vechain server,  open “Channels & Roles” and select “Development on vechain”.  “BUILDER COMMUNITY” will show on the side bar.

## Conclusion

VeChain Improvement Proposals (VIPs) are a way for the community to propose and discuss improvements to the VeChain ecosystem. By following the VIP guidelines, anyone can contribute to the development of the VeChain blockchain.
