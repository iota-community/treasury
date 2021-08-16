**NOTE: THIS DOCUMENT IS VERY MUCH A WORK IN PROGRESS.**

# Overview

This specification summarizes requirements to develop the technology to be used in the initial vote to burn the tokens or form the IOTA Community Treasury.

Assuming the decision is made to form a Community Treasury, this voting system can be adapted to support the early operations of the newly created Community Treasury.

The following overview highlights various use cases and high-level technical designs combined with mock-ups of the proposed system.
A more detailed design will follow once requirements,dependencies and scope are refined and agreed upon. 

Flexibility is important at this stage. Therefore the initial design provides the opportunity to evaluate various technical approaches that could be prototyped and decided upon in the future.The intent is to encourage a more agile process as we move forward.

**Voting tech consists of three parts:**
- Hornet Node plug-in. See RFC: https://github.com/WernerderChamp/protocol-rfcs/blob/master/text/0000-chrysalis-referendum/0000-chrysalis-referendum.md
- DAO Website
- Firefly plug-in

The requirements detailed below were established based on numerous discussions and meetings that took place on IOTA's Discord #governance and #voting-tech channels. 
#Governance meetings took place on Thursdays 4pm CEST.

You can find various documented meeting notes in the governance [Github repository](https://github.com/iota-community/Community-Governance/tree/main/meetings). 

Finally, many conversations and  meetings with several community members took place to help develop the approach.


> **Once this document is finalized**; it should be scrutinized by the community and agreed upon. This proposal **"could"** essentially become a requirements document for the voting tech that will be need to be built. Thus establishing the base specification.

Key caveats:
>   The Hornet node plug-in is (supposed to be) built by the Hornet team.
>   
>    The other two-parts are still to be decided. 
>   
Finalizing this document **"will"** speed up the process of finding the right team(s) to build the voting tech. 

We are flexible on the approach taken in this proposal and are open to discuss alternative strategies.

## How to propose changes in this document
Please submit your suggestions via pull request within [this](https://github.com/iota-community/Community-Governance) repository.

## TODO
- [ ] Initiate process to agree to a domain name
- [ ] For the DAO with ISCP this will need to be extended into a full forum style discussion board where the community can communicate with the proposal submitters and discuss all topics related to the proposals. Phylo what was this about?
- [ ] Finalize Firefly mock-ups
- [ ] DRAFT Web DAO mock-ups
- [ ] Hornet Node plugin - this needs to be still worked out more -  data sizes are too big; IMHO 1 MB proposal size is not useful. This is morelikely for the other RFC.

### IDEAS on UX/UI 
* Proposals on DAO Website could be similar to Github Discussions: https://github.com/iota-community/Community-Governance/discussions
* Firefly designs link: 

## Terms
- [Hornet](https://github.com/gohornet/hornet) - Community driven IOTA Full Node- [Hornet Plugin](https://github.com/gohornet/hornet/tree/main/plugins) - Extension within Hornet that can be used by various node operators
- [Firefly](https://github.com/iotaledger/firefly) - IOTA's official wallet 
- Proposal - Proposal to be voted on by IOTA's token holders
- Question - Each proposal can consist of multiple question and each of them can have various voting options.
- Voting Option - Option that can be voted on within the question.
- Vote - A casting vote by IOTA token holder.
- [ISCP](https://github.com/iotaledger/wasp) - IOTA Smart Contract Platform

## Key design decisions

It is desired to get this tech up and running ASAP. The intent is to avoid building an overly complex system that will require several modifications over time. 

For example, many parts might change due to the [ISCP](https://github.com/iotaledger/wasp) introduction and the DAO intends to start voting ASAP. 

Once ISCP is introduced on IOTA's Mainnet, we should move away from some parts highlighted below in the system (i.e. Github) and use ISCP instead. Ideally, we could do this without a significant impact to the users. 

Lastly, it is essential for the community to start making decisions ASAP if we are going ahead to BUILD or BURN treasury funds, as this can significantly impact IOTA's future adoption.

### Github
We envision IOTA's Community Governance Github repository to be utilized for proposal management. This will serve as a staging area for proposals prior to  their final submission to the community nodes and Firefly.

Github is considered a trustworthy source and provides a fairly secure environment for proposals at this stage. 

This method should be sufficient for our initial rollout as we await the release of ISCP.

The benefits of this approach is that it provides enough transparency into the process and allows everyone to participate.

### Hornet Node plugin
As specified within the RFC, participating node operators can utilize the Hornet plugin to provide critical functions to support the voting process. Such as: 
- Track available proposals for the vote, their status, and progress
- provide API endpoints for users and applications to fetch this data from the nodes
- Ability to count the issued votes based on token amount, holding period and stated opinion on UTXO's and produce the final results of every vote out of this counting process.

### Website
The Website should be a simple single page application (React, Angular, Vue, etc.) that sources data from the these two sources:
- Github
- Hornet Node

Proposal management and voting will be managed within the two listed above and the website. It's a simple UI design on the top to present the information in a friendlier fashion to help users better understand the process.

Additionally, it could be leveraged to provide an introduction to the voting process and provide tutorials for the user on how to manage his/her voting attempt.

> Potentially, the Website can integrate with Github and allow proposal management within the application (i.e. simplify it for users without Github knowledge). This can help transition to ISCP as users do not need to get used to the new system, and it'll be just underlying tech that changes.
> 
> We could build the Website with Docosaurus similar to the Wiki System. The rich text editor developed by Jeroen could be an optimal tool to make it easy for users to create proposals and submit them as pull requests to the GitHub Repo. The plugin is designed to do exactly this so that we could leverage this tool. The structure of a proposal would be predefined in a template and the user will be able to fill in the fields with the required information.

## Firefly plugin
The extension for Firefly will support the ability to view proposals that are "available for a vote", "in progress proposals", and "finished proposals". 
Main purpose is the ability for the user to cast their vote, change their vote and see votes of other users (generalized).

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
* The pull request enters a defined, required approval process, and awaits all approvals (mixture of IF/community members)
    * This can be automated via GitHub Actions inc. various verification. 
    * Approvers needs to be "elected or approved" by the community. We envision this approval by the absence of objection against the proposed approvers via CODEOWNERS
* **(TBD)** approval opens a pull request and decide to approve/reject based on numerous factors:
    * number of votes within pull request?
    * number of people monitoring pull request?
    * number of follow-ups?
    * tbd
### Proposal Approval
* Once the pull request is approved, it'll get merged into the master branch under the "proposal" folder. 
    * GitHub Action can be used to publish the proposal
        * iotaledger/gh-tangle-release can be used to automatically create an immutable hash of the release and post it as a message in the tangle.
    * GitHub Action can also post information about a new proposal to the #governance-discussion channel on Discord
* Any new proposal within the "proposals" master branch are picked and promoted within Hornet nodes (with voting plugin) & Firefly
    * Upload to nodes is done manually. *Ideally, we can build a tool which can automate this and node operators can enable this if they want to.*
### Proposal Termination
* **(TBD)** Do we want to handle it, how, when? Amendment to an existing proposal with new file "proposal" folder in master branch?
* During the **"build phase"** we could limit keep this in the hands of the Repo Maintainers and not allow any random proposals

## Firefly
### View Proposals
* User navigates into the voting tab
* User can view existing proposals (only those approved from the GitHub Master Branch, sorted by commencing date).
    * Note, Firefly gets proposals from the master branch proposals/ folder and check if it exists on the node. There might be a scenario where it's not yet uploaded to nodes yet.
      * Firefly must periodically check for new proposals / at the start
* By default, users only see upcoming/in progress proposals
* They can filter and see ended ones (history)
* User should be able to easily identify proposals still pending their vote 
* User can easily open proposal on the DAO Website to access fully history (i.e. pre-approval)
### Submit Vote
* User selects the proposal which is still pending their vote
* User votes on a voting option within each question of the proposal
  * User must understand voting power (in IOTA units) and "weight" of their vote as it might be different: 
    * If it's delegated vote, how much have been delegated to them?
    * If proposals are already in "Holding" state
    * If they change their vote during "Holding" stage of the proposals?
* User can select the wallet they want to vote with (amount of funds to allocate as opinion)
* User can open their vote on the Web DAO and from there they have an ability to share their vote options online (discord/Twitter/etc.). This could potentially include link to their actual "vote" other users can view within DAO Website.
### Change Vote
* User opens commencing/holding proposals and can view their existing vote
* They can change their vote
  * Understand new weight of their vote
* They get the direct information of the impact of the change (i.e "due to the change you have currently gained XXX voting power for option X and now will continue to build up voting power on option Y")
* They share their vote easily online again
### View Vote Status
* User is able to view status and progress of each proposal and their own vote
* Ability to view overall results 

## DAO Web
### Home page
* Ideally starts with video what DAO is all about
* Continues with more detailed explanation
* Ends with "cast your vote" / "create a proposal"

### Proposals
* Ability to view all proposals (drafted/submitted/approved/commencing/holding/ended)
  * Ability to navigate to any proposal including historic one.
* Widget to show currently commencing/holding proposals
* Ability to draft a new proposals
    * This could be an explanation how to do it in Github
    * Or we build integration on top of the Github with the markdown editor that creates pull requests - ideal! (Docosaurus, etc.)
* Ability to cast vote and Firefly opens automatically (deep link)

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
* About DAO and it's history
* Members that approve proposals within GitHub (members of iota-community/treasury ?)

### Firefly Voting Logic
![](https://i.imgur.com/Fu9Npwg.png)

# High-level Technical Requirement
In this section we highlight some of the key technical assumptions. This can be further expanded or changed down the track as required during the implementation. It's explained to help the builder understand mechanics and potentially direct them on possible approach.

In summary, Firefly code changes should be minimal and should be primarily focused on ability "execute the vote". Any other functionality should be done within Web DAO. This will reduce security risks and allows us to quickly build up more features. 

## Middleware for proposal management
> Ideally, middleware is only an intern solution before ISCP are implemented and ready to be used.

Middleware management tool will have to build to proxy Github's list of proposal to Firefly and Web DAO. This should be a very simple proxy so it can be easily audited for any security issue.

Proxy will keep a clone of Github repository within database that clients can use to get the latest. Following tool can be used to achieve that: https://isomorphic-git.org.

One approach could be to utilise Google's Cloud Firestore database accessed by Firefly/Web DAO. It also supports websocket for very efficient communication. The Cloud Firestore database instance can also be used by Web DAO to manage proposals and other added functionality. 

Firebase Functions can be utilized to execute actions based on changes within the database as well as on monitoring changes within the Github repository. 

### Firebase Auth
When authentication to Github is required within Web DAO Firebase AUTH can be utilized for that. 

> Note, Firebase product is not necessary and it's an example of tool that could be utilized to deliver globally scalable solution that supports above requirements.

## Historical Access
It's very important that all voting data are persisted for long period of time to provide constant transparency into the decision made since the day one. Ideally, group of trusted perma-nodes should eventually be established compare to just storing the voting result in a repository as this data could be manipulated. User can see data from various permanodes within Web DAO and whether they're consistent. DAO could potentially fund infrastructure cost for these permanodes. 

**Voting permanode key requirements:**
- store complete transaction history for every vote on each proposal
- provides list of all proposals
- ability to retrieve voting results / transactions per referendum

> Firefly does not connect to any permanode at the moment and we ideally should not change that. Firefly will only know information due user's action (stored in Stronghold) or available within Hornet Node (i.e. proposals voting overview)


### Wallet.rs does not require any enhancements/changes
As per our understanding wallet.rs does not require any changes as it already supports "no value" transactions.

### External links in Firefly
Any external links in Firefly must be whitelisted and clearly defined. Ideally, only connection to "middleware" and Web DAO should be allowed.

# Known Limitations / Assumptions
## Firefly deep links
Should we supported soon and in time for WEB DAO.

## Firefly mobile
Firefly mobile wallet will not be supported as it's still in conceptual stage and it's long away from delivery. 

## IOTA DID will not be implemented within the phase 1
It was proposed that IOTA DID could be used to authenticate users within the Web DAO and used for proposal management. This is still evolving technology (limited tooling) and it's preferred to wait to reduce possible dependency.

## Privacy during the voting
Firefly voting plugin will behave same way as Firefly and it'll group all addressees within the wallet. This means if user has multiple addresses with balance within one Firefly wallet those would be grouped together. Firefly applies same approach when funds are send from an existing wallet.

It's understand that some users might prefer to avoid that for privacy reasons. Those users will have to use different tool and not Firefly.

**Suggested solutions**
1. We discussed an approach where Firefly could potentially spread the vote over a time although this would be an extensive time period that could cause a security risk. It would also potentially have low adoption as users will not be comfortable to keep their wallet open.
2. Node could potentially spread transactions. Although, this would require user trusting the node and create yet another problem.
3. Delegation of the vote. User might be able to create a new wallet and delegate their other wallets to this new address over longer period. Use this new wallet to vote. This approach can create another problem as we might have to depend on perma-nodes and figure the validity of their data.

> For those users that are concerned about privacy it might also be recommended to use VPN in case Hornet node logs usage.

## Firefly changes
Firefly does not yet support plug-in system and it is at research stage. Therefore voting plugin will be developed as new core function within the Firefly wallet and IF will drive UX/UI design and overall architecture.

## Transition to ISCP
Web DAO / Firefly should potentially allow relatively simple transition to ISCP and limit impact to the user as well as reduce the time for delivery.

> Ideally, there is no impact to user and we would just change the "engine" but keep the same dashboard and steering wheel. 

# Contributors
* @adamkundrat
* @Phyloiota
* @GmanRedmond
