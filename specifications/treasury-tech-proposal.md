**NOTE: THIS DOCUMENT IS VERY MUCH A WORK IN PROGRESS.**

# Overview

This specification summarizes requirements to develop the technology to be used in the initial vote to burn the tokens or form the IOTA Community Treasury.

Assuming the decision is made to form a Community Treasury, this voting system can be adapted to support the early operations of the newly created Community Treasury Project.

We find it important to highlight that the proposed handling and approving mechanisms here are not meant to be implemented for the final stage of a Treasury (DAO etc). The way such a future system will handle proposals and decisions will have to be decided by the community.

The purpose of this specification is to establish 2 things:
- Build the tools that the community needs to make the first important decision
- If the outcome of this first decision is to continue and build a Treasury management system, give the people that work on building this system a way to involve the community in the building process (through proposals / comments) when it comes to directional decisions of how they should work.

The following overview highlights various use cases and high-level technical designs combined with mock-ups of the proposed system.
A more detailed design will follow once requirements, dependencies and scope are refined and agreed upon. 

Flexibility is important at this stage. Therefore the initial design provides the opportunity to evaluate various technical approaches that could be prototyped and decided upon in the future. The intent is to encourage a more agile process as we move forward.

**Voting tech consists of three parts:**
- Treasury Website to inform the community
- Firefly plug-in to display referendums to the user and enables the user to cast votes
- Hornet Node plug-in that counts votes in the IOTA Tangle and produces verifiable results of the vote. Developing this plugin is not in the scope of this specification.

The tools (Firefly and Hornet plugin) should work with the referendum structure proposed in this RFC: https://github.com/WernerderChamp/protocol-rfcs/blob/master/text/0000-chrysalis-referendum/0000-chrysalis-referendum.md (not yet finaly defined)


The requirements detailed below were established based on numerous discussions and meetings that took place on IOTA's Discord #governance and #voting-tech channels. 
#Governance meetings took place on Thursdays 4pm CEST.

You can find various documented meeting notes in the governance [Github repository](https://github.com/iota-community/Community-Governance/tree/main/meetings). 

Finally, many conversations and meetings with several community members took place to help develop the approach.


Key caveats:
>   The Hornet node plug-in is (supposed to be) built by the Hornet team.
>   
>    The other two parts are still to be decided. 
>   
Finalizing this document will speed up the process of finding the right team(s) to build the voting tech. 

We are flexible on the approach taken in this proposal and are open to discuss alternative strategies.




## Terms
- [Hornet](https://github.com/gohornet/hornet) - Community driven IOTA Full Node- [Hornet Plugin](https://github.com/gohornet/hornet/tree/main/plugins) - Extension within Hornet that can be used by various node operators
- [Firefly](https://github.com/iotaledger/firefly) - IOTA's official wallet 
- Proposal - Proposal to be voted on by IOTA's token holders
- Question - Each proposal can consist of multiple questions, and each of them can have various voting options.
- Answer - Option that can be voted on within the question.
- Vote - A casting vote by IOTA token holder.
- [ISCP](https://github.com/iotaledger/wasp) - IOTA Smart Contract Protocol

## Key design decisions

It is desired to get this tech up and running ASAP. The intent is to avoid building an overly complex system that will require several modifications over time. 


Once ISCP is introduced on IOTA's Mainnet, we should move away from some parts highlighted below in the system (i.e. Github) and use ISCP instead. Ideally, we could do this without a significant impact on the users. 

Lastly, it is essential for the community to start making decisions ASAP if we are going ahead to BUILD or BURN treasury funds, as this can significantly impact IOTA's future adoption.

### Github
We envision IOTA's Community Governance Github repository to be utilized for proposal management in the phase of building the Treasury management System for the community. This will serve as a staging area for proposals prior to their final submission to the community nodes and Firefly.

Github is considered a trustworthy source and provides a fairly secure environment for proposals at this stage. 

This method should be sufficient for our initial rollout as we await the release of ISCP.

The benefits of this approach are that it provides enough transparency into the process and allows everyone to participate.

### Hornet Node plugin
As specified within the RFC, participating node operators can utilize the Hornet plugin to provide critical functions to support the voting process. Such as: 
- Track available proposals for the vote, their status, and progress
- provide API endpoints for users and applications to fetch this data from the nodes
- Ability to count the issued votes based on token amount, holding period and stated opinion on UTXO's and produce the final results of every vote out of this counting process.

### Website
The Website should be a simple single page application (React, Angular, Vue, etc.) that sources data from these two sources:
- Github
- Hornet Node

It's a simple UI design to present the information in a friendlier fashion to help users better understand the process.

Additionally, it could be leveraged to provide an introduction to the voting process and provide tutorials for the user on how to manage his/her voting attempt.

> Potentially, the Website can integrate with Github and allow proposal management within the application (i.e. simplify it for users without Github knowledge). This can help transition to ISCP as users do not need to get used to the new system, and it'll be just underlying tech that changes.


### Firefly plugin
The extension for Firefly will support the ability to view proposals that are "available for a vote", "in progress proposals", and "finished proposals" (the solution for storing finished proposals needs to be discussed further). 
Main purpose is the ability for the user to cast their vote, change their vote and see the votes of other users (generalized).

# Detailed Usage

## Github
### Draft/Request for Comments Proposal
* A user creates a new pull request with a proposal (DRAFT) specification against the governance repository master branch
    * new file in proposals/<hash_id_of_content>.json forked from iota-community/treasury 
* The user provides sufficient information within this proposal following the proposal template described in this [Proposal RFC](https://github.com/WernerderChamp/protocol-rfcs/blob/master/text/0000-chrysalis-referendum/0000-chrysalis-referendum.md)
* The created pull request gets shared within the community to provide feedback/comments/etc.
* The proposer should make sure to gather a lot of interest and promote their idea before it's passed to the submission stage.
### Submit Proposal
* Once the proposer is comfortable to proceed, they will need to submit their proposal for approval by moving the created pull request from "Draft"- status to "Ready for Review"
* The pull request enters an approval process, and awaits all approvals (mixture of IF/community members)
    * This can be automated via GitHub Actions inc. various verification. 
    * Approvers need to be "elected or approved" by the community. We envision this approval by the absence of objection against the proposed approvers via CODEOWNERS

### Proposal Approval
* Once the pull request is approved, it'll get merged into the master branch under the "proposal" folder. 
    * GitHub Action can be used to publish the proposal as a "release"
        * iotaledger/gh-tangle-release can be used to automatically create an immutable hash of the release and post it as a message in the tangle.
    * GitHub Action can also post information about a new proposal to the #governance-discussion channel on Discord
* Any new proposal within the "proposals" master branch can be integrated into Firefly via regularly API calls
* The way hornet integrates new proposals needs to be defined by the hornet team, but it might also be a sufficient way to use the provided API access.
  
### Proposal Termination
* During the building phase of the system we can move proposals to a "finished" folder if the vote has occured. Also proposals that dont get approved should be moved in an "rejected" folder and so still be publicly available and visible.



## Firefly
### View Proposals
* User navigates into the voting tab
* User can view existing proposals which Firefly has uploaded via the API call from the Github release.
    * Note, Firefly gets proposals from the master branch proposals/ folder and check if it exists on the node. There might be a scenario where it's not uploaded to nodes yet. We will have to define how to handle this situation. So Firefly would need to communicate with its connected node if the node is aware of the proposal
      * Firefly must periodically check for new proposals and at Application start
* By default, users only see upcoming/in progress proposals
* They can filter and see ended ones (history) 
     * Note: This needs to be discussed and defined from where this data will be sourced, or if Firefly stores it internally / locally
* User should be able to easily identify proposals still pending their vote 
* User can easily open proposal on the Treasury Website to access fully history (i.e. pre-approval) via a provided link (button etc)
### Submit Vote
* User selects the proposal which is still pending their vote
* User votes on an answer within each question of the proposal
* User must be informed about voting power (in IOTA units) and "weight" of their vote as it might be different: 
    * If proposals are already in "Holding" state and therefore the user cannot reach 100% voting power anymore
    * If they change their vote during "Holding" stage of the proposals and therefore voted for 2 different options with the specific voting power that is related to the holding time of each option
* User can select the wallet they want to vote with (amount of funds to allocate as opinion)
* User can open their vote on the Treasury Web and from there they have an ability to share their vote options online (discord/Twitter/etc.). This could potentially include a link to their actual "vote" other users can view within Treasury Website.
* To decouple the Voting from "spending funds" we envision to not display any value metrics in the voting Tab. We would always refer to IOTA votes and not display USD value or call it tokens.
### Change Vote
* User opens already processed proposals and can view their existing vote
* They can change their vote
  * Give full information to understand the new weight of their vote
* They get the direct information of the impact of the change (i.e "due to the change you have currently gained XXX voting power for option X and now will continue to build up voting power on option Y")
* They share their vote easily online again
### View Vote Status
* User is able to view status and progress of each proposal and their own vote
* Ability to view overall results / ongoing counts

## Treasury Web
### Home page
* Ideally starts with video what the vote is all about
* Continues with more detailed explanation
* Ends with "cast your vote" / "create a proposal"

### Proposals
* Ability to view all proposals (drafted/submitted/approved/commencing/holding/ended)
  * Ability to navigate to any proposal including historic ones.
* Widget to show currently commencing/holding proposals
* Ability to draft new proposals
    * This could be an explanation of how to do it in Github
    * Or we build integration on top of the Github with the markdown editor that creates pull requests - ideal! (Docosaurus, etc.)
* Ability to cast a vote and Firefly opens automatically (deep link) needs to be discussed


#### View Individual proposal
* Current status and all it's attributes
* If active/ended
    * User can see all votes by individual users with linked to explorer
    * Overall progress of the vote
    * Reported numbers by various nodes
    * Ability to cast a vote and Firefly opens automatically
  
### Subscribe to news
* Email
* Twitter
* Discord

> Email database might not be ideal as it might be hacked.

### About
* About the Tokens, the Treasury and it's history
* Display Members that approve proposals within GitHub (members of iota-community/treasury ?)

### Firefly Voting Logic



### IDEAS on UX/UI 
* Proposals on Treasury Website could be similar to Github Discussions: https://github.com/iota-community/Community-Governance/discussions and provide a forum for the community to discuss the topic

# High-level Technical Requirement
In this section we highlight some of the key technical assumptions. This can be further expanded or changed down the track as required during the implementation. It's explained to help the builder understand mechanics and potentially direct them to a possible approach.

In summary, Firefly code changes should be minimal and should be primarily focused on the ability to "execute the vote". Any other functionality should be done within Treasury Website. This will reduce security risks and allows us to quickly build up more features. 

## Middleware for proposal management
> Ideally, middleware is only an intern solution before ISCP is implemented and ready to be used.

Middleware management tool will have to build to proxy Github's list of proposals to Firefly and Treasury Website. This should be a very simple proxy so it can be easily audited for any security issue.

The proxy will keep a clone of the Github repository within a database that clients can use to get the latest. The following tool can be used to achieve that: https://isomorphic-git.org.

One approach could be to utilize Google's Cloud Firestore database accessed by Firefly/Treasury Web. It also supports WebSocket for very efficient communication. The Cloud Firestore database instance can also be used by Treasury Web to manage proposals and other added functionality. 

Firebase Functions can be utilized to execute actions based on changes within the database as well as on monitoring changes within the Github repository. 

### Firebase Auth
When authentication to Github is required within Web DAO Firebase AUTH can be utilized for that. 

> Note, Firebase product is not necessary and it's an example of tool that could be utilized to deliver a globally scalable solution that supports above requirements.

## Historical Access
It's very important that all voting data must be persisted for a long period of time to provide constant transparency into the decision made since day one. Ideally, a group of trusted perma-nodes should eventually be established compared to just storing the voting result in a repository as this data could be manipulated. Users can see data from various permanodes within Treasury Web and whether they're consistent. Treasury could potentially fund infrastructure costs for these permanodes. 

**Voting permanode key requirements:**
- store complete transaction history for every vote on each proposal
- provides a list of all proposals
- ability to retrieve voting results / transactions per referendum

> Firefly does not connect to any permanode at the moment and we ideally should not change that. Firefly will only know information due user's action (stored in Stronghold) or available within Hornet Node (i.e. proposals voting overview)


### Wallet.rs does not require any enhancements/changes
As per our understanding wallet.rs does not require any changes as it already supports "no value" transactions.

### External links in Firefly
Any external links in Firefly must be whitelisted and clearly defined. Ideally, the only connection to "middleware" and Treasury Web should be allowed.

# Known Limitations / Assumptions
## Firefly deep links
Should be supported soon and in time for Treasury Web.

## Firefly mobile
Firefly mobile wallet will not be supported as it's still in the conceptual stage and it's long away from delivery. 

## IOTA DID will not be implemented within phase 1
It was proposed that IOTA DID could be used to authenticate users within the Web DAO and used for proposal management. This is still evolving technology (limited tooling) and it's preferred to wait to reduce possible dependency.

## Privacy during the voting
Firefly voting plugin will behave same way as Firefly and it'll group all addressees within the wallet. This means if a user has multiple addresses with balance within one Firefly wallet those would be grouped together. Firefly applies the same approach when funds are sent from an existing wallet.

We understand that some users might prefer to avoid that for privacy reasons. Those users will have to use different tools and not Firefly.

**Suggested solutions**
1. We discussed an approach where Firefly could potentially spread the vote over time although this would be an extensive time period that could cause a security risk. It would also potentially have low adoption as users will not be comfortable keeping their wallet open.
2. Node could potentially spread transactions. Although, this would require the user to trust the node and create yet another problem.
3. Delegation of the vote. A user might be able to create a new wallet and delegate their other wallets to this new address over longer period. Use this new wallet to vote. This approach can create another problem as we might have to depend on perma-nodes and figure the validity of their data.
4. Concerned users could spread voting over several voting instances manually and so avoid any connection between their addresses

> For those users that are concerned about privacy it might also be recommended to use VPN in case Hornet node logs usage.

## Firefly changes
Firefly does not yet support a plug-in system and it is at the research stage. Therefore voting plugin will be developed as a new core function within the Firefly wallet and IF will drive UX/UI design and overall architecture.

## Transition to ISCP
Web DAO / Firefly should potentially allow a relatively simple transition to ISCP and limit the impact to the user as well as reduce the time for delivery.

> Ideally, there is no impact to the user, and we would just change the "engine" but keep the same dashboard and steering wheel. 


## How to propose changes in this document
Please submit your suggestions via pull request within [this](https://github.com/iota-community/Community-Governance) repository.

## TODO
- [ ] Initiate process to agree to a domain name
- [ ] For a potential DAO with ISCP, this will need to be extended into a full forum style discussion board where the community can communicate with the proposal submitters and discuss all topics related to the proposals.
- [ ] Finalize Firefly mock-ups
- [ ] DRAFT Treasury Web mock-ups
- [ ] Hornet Node plugin - this needs to be still worked out more -  data sizes are too big; IMHO 1 MB proposal size is not helpful. This is more likely for the other RFC.


# Contributors
* @adamkundrat
* @Phyloiota
* @GmanRedmond
