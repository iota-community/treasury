+ Feature name: chrysalis-referendum
+ Start date: 2021-07-11
+ RFC PR: To be created...

# Summary

This RFC defines On-Tangle voting for Chrysalis Phase 2. Specifically, it describes the structure and flow of a referendum, the format of voting data to be included in the transaction, and all endpoints on the node side.

# Motivation

[With the IOTA Foundation offering the community the opportunity to decide on the fortune of the unclaimed tokens](https://blog.iota.org/iota-community-treasury-and-genesis-validation), IOTA needs to provide a system for community members to create and vote on referenda. In community calls, we figured out that the only way to achieve a fair referendum without investing years in R&D would be a simple token-based system. With this idea in mind, any token holders can use their tokens to vote for one of the options on each question, one token, one vote. During the weekly meetings, the majority of the community agreed to have further ongoing community referenda to establish a system and decide on the features of this system through voting. Therefore the system also needs to support future referenda, as well as multiple at the same time.

The system describes below tries to address the following points:

* No protocol fork is required as voting data is stored in the data (indexation) payload of a value transaction
* Protection against flash loan attacks, where an attacker buys a lot of tokens to game referenda and dumps them instantly after the referenda
  * This is achieved by adding a third factor to voting weight, which is holding time
* No token locking required
* Referendums can have multiple questions (allows compact voting without much overhead)
* Tokens remain in your wallet at all times


# Detailed design

## Referendum flow

A referendum has 4 different stats

### Upcoming

Any referendum will start here. The creator specifies the questions and all answer options and the beginning, start of the holding phase, and end of the referendum (see next section).

Referendums need to be published as a Draft Pull request in the Treasury Github Repository. From this time on, the community can comment on the pull request, start discussions or issues, and request changes as commits to the PR. As soon as all those requests have been addressed, the Pull Request needs to be checked and the correct format of the Data verified by the Reviewers.

Those reviewers need to be elected, announced, or agreed on by the community. The Reviewers need to agree on the correct form of the Referendum. hen all reviews are submitted and agreed on, an ID is created by hashing the entire referendum data. Therefore the same combination always results in the same ID. Now the Referendum can be merged into the Master branch.

As soon as a new Referendum is merged into the Master branch, the Firefly Wallet can upload it using the existing call to IF's amazon S3 bucket where it also normally checks for new releases of the hornet software. In this way the system can leverage on existing infrastructure from IF.

Referendums are neither automatically added to all nodes, nor are they broadcast on the network. Instead, the referendum must be added to multiple nodes manually. Because submitting the same referendum to multiple nodes always results in the same ID, it is possible to compare the data of multiple nodes. This also provides security, as users can verify they are voting for the correct referendum (e.g. because the ID is published on Github).
It needs to be discussed if the Hornet plugin can use the Github API call instead of the manual upload.

The creator of a referendum needs to set the voting start milestone at least a few days into the future because a Referendum can only be added to nodes before the commencing phase starts.

Firefly can always add referendums to the wallet as long as the phase is "Commencing" or "Holding."

### Commencing

Once the starting milestone defined in the referendum has been reached, the referendum enters the commencing phase. During this phase, votes can now be cast in the Firefly Wallet and are recorded by the node, which constantly updates the number of tokens voting for every option on each question whenever a milestone comes in. However, the votes still have no weight.

Whatever time is set as the beginning, there will always be a place in the world where it will be very inconvenient (e.g., the middle of the night). This phase grants fairness to every timezone, as it does not matter when you submit your vote in this phase. Also, it can be used to draw public attention and spread the word. Ideally, this phase should last for a few days or even a week, so everyone has a chance to give their vote 100% of its weight.

The node would ignore any votes for a referendum that is still upcoming or unknown. Firefly will only display such Referendums that have entered the commencing phase and therefore not allow such votes. Users could still try to cast votes before this by using other clients. Thus the node will ignore them until the beginning milestone is reached. This does not cause the validation of referenda to fail. For example, if someone votes for three referenda at once but only one is known to the node, it will count the vote usually for one and ignore the other two.

### Holding

After the commencing phase is over, the holding phase will begin. Every time a milestone comes in, the node will first update the number of tokens voting for each option on every question (if there were changes in the ledger) and then add this amount to the total count for each option. The addition always adds weight for the following period since token balances cannot change until a new milestone is received. It will happen first on the Holding Start Milestone, and the last time one milestone before the vote ends.

This adds a time factor to the vote. If you buy tokens and vote during the ongoing referendum, you only get weight for the remaining time. Similarly, if you decide to sell, you will miss out on the weight for the remaining time. If you choose to change your opinion, your old opinion will get the weight of the elapsed time since the holding phase started, while your new opinion will get the weight of the remaining time.

Example: Assume Alice has 2i and does not vote at all. Now she sends her tokens to Bob 10000 milestones after the beginning of the holding period (approx. 1d 4h). Bob instantly votes for option A. After another 20000 milestones have passed, he sends the tokens to Charlie, who instantly votes option B. 5000 Milestones later, the referendum ends. Assuming the total supply consists of 2 iotas the outcome would be:
*     2i*10000=20000 are not counted in the voting
*     2i*20000=40000 for Build (80%)
*     2i* 5000=10000 for Burn (20%)
*     Turnout: 50000/70000=71.4% 

In reality, those numbers would be a lot higher, enough to exceed the 64-bit integer limit given enough participation.

### Ended

Once the end milestone has been received, the referendum is over. New votes are ignored, just as if the node did not know the referendum at all. 

## Referendum format

Users can submit referenda to any nodes that have the endpoint open. A referendum consists out of one or multiple questions, which each has various options to choose from. It also has three milestone numbers stored:
* when the referendum starts
* when holding starts
* when the referendum is over. 
The referendum, the question, and each answer have an information field, which may contain additional information about it.

### Detailed format

<table>
    <tr>
        <th>Name</th>
        <th>Type</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>Referendum Name Length</td>
        <td>uint8</td>
        <td>
        The length of the referendum name in bytes
        </td>
    </tr>
      <tr>
        <td>Referendum Name</td>
        <td>String</td>
        <td>
        The referendum question in UTF-8 format
        </td>
    </tr>
      <tr>
        <td>Begin Milestone</td>
        <td>uint32</td>
        <td>
        The milestone where the referendum begins
        </td>
    </tr>
    </tr>
      </tr>
      <tr>
        <td>Holding Start Milestone</td>
        <td>uint32</td>
        <td>
        The milestone where the referendum enters holding phase
        </td>
    </tr>
    </tr>
      </tr>
      <tr>
        <td>End Milestone</td>
        <td>uint32</td>
        <td>
        The milestone where the referendum ends
        </td>
    </tr>
    <tr>
        <td>Question Count</td>
        <td>uint8</td>
        <td>The count of questions proceeding</td>
    </tr>
    <tr>
        <td valign="top">Voting question <code>oneOf</code></td>
        <td colspan="2">
            <details open="true">
                <summary>Voting question</summary>
                <blockquote>
                A question that can be voted on
                </blockquote>
                <table>
                    <tr>
                        <td><b>Name</b></td>
                        <td><b>Type</b></td>
                        <td><b>Description</b></td>
                    </tr>
                    <tr>
                        <td>Question Idx</td>
                        <td>uint8</td>
                        <td>
                        The index of the question. Counted from 1 to 255.
                        </td>
                    </tr>
                    <tr>
                        <td>Question Text Lengh</td>
                        <td>uint8</td>
                        <td>
                        The length of the question in bytes
                        </td>
                    </tr>
                    <tr>
                        <td>Question name</td>
                        <td>String</td>
                        <td>
                        The text of the question in UTF-8 format
                        </td>
                    </tr>
                    <tr>
                        <td>Answer Count</td>
                        <td>uint8</td>
                        <td>
                        The amount of answers following
                        </td>
                    </tr>
                      <td valign="top">Voting Answer <code>oneOf</code></td>
                      <td colspan="2">
                       <details open="true">
                          <summary>Voting answer</summary>
                           <blockquote>
                          A votable answer for the question
                           </blockquote>
                           <table>
                               <tr>
                                <td><b>Name</b></td>
                                <td><b>Type</b></td>
                                <td><b>Description</b></td>
                               </tr>
                               <tr>
                                <td>Answer ID</td>
                                <td>uint8</td>
                                <td>
                                The ID of the answer. Counted from 1 to 255.
																													 		</td>
																															</tr>
																															<tr>
																																<td>Answer Text Length</td>
																																<td>uint8</td>
																																<td>
																																The length of the answer text in bytes
																																</td>
																															</tr>
																															<tr>
																																<td>Answer Text</td>
																																<td>String</td>
																																<td>
																																The text of the answer in UTF-8 format
																																</td>
																															</tr>
																															<tr>
																																<td>Answer Additional Information Length</td>
																																<td>uint16</td>
																																<td>
																																The length of the answer additional information text in bytes
																																</td>
																															</tr>
																															<tr>
																																<td>Answer Additional Information</td>
																																<td>String</td>
																																<td>
																																Additional information about the Answer UTF-8 format
																																</td>
																															</tr>
																														</table>
																												</details>
																										</td>
                        <tr>
                        <td>Question Additional Information Length</td>
                        <td>uint16</td>
                        <td>
                        The length of the question additional information text in bytes
                        </td>
                    </tr>
                    <tr>
                        <td>Question additional Information</td>
                        <td>String</td>
                        <td>
                        Additional information about the question in UTF-8 format
                        </td>
                    </tr>
                  </table>
          </details>
  </td>
    <tr>
        <td>Referendum Additional Information Length</td>
        <td>uint16</td>
        <td>
        The length of additional information in bytes. Set to 0 if empty
        </td>
    </tr>
      <tr>
        <td>Referendum Additional Information</td>
        <td>String</td>
        <td>
        Additional information in UTF-8 format. Field does not exist if the length was set to 0
        </td>
    </tr>
</table>

<i>The BLAKE2b-256 hash of this object is the referendum ID, which is used as the UUID of the referendum</i>

### Syntactical validation

* The entire object must not exceed 1 million bytes
* The referendum must start in 1 ≤ x ≤ 600000 milestones (~10 weeks)
* The holding period has to start 360 ≤ x ≤ 600000 milestones after the referendum started
* The referendum has to end 360 ≤ x ≤ 600000 milestones after holding started
* All additional information fields must not exceed 1000 bytes
* Questions must be ordered in ascending order by their index
* Answers must be ordered in ascending order by their ID

## Voting Format

Voting data will be sent via the indexation payload of a value transaction. To vote, you send a transaction to yourself, specifying your decisions.

### Criteria for a valid voting transaction

* Must be a value transaction
* Inputs must all come from the same address. Multiple inputs are allowed
* Has a singular output going to the same address as all input addresses.
  * Output Type 0 (SigLockedSingleOutput) and Type 1 (SigLockedDustAllowanceOutput) are both valid for this
* The Index must be "IOTAVOTE"
* The data must be parseable

### Structure of the voting data

<table>
    <tr>
        <th>Name</th>
        <th>Type</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>Referendum count</td>
        <td>uint8</td>
        <td>
        The amount of referenda following. Must be at least one
        </td>
    </tr>
    <tr>
        <td valign="top">Referendum vote<code>oneOf</code></td>
        <td colspan="2">
            <details open="true">
                <summary>Referendum vote</summary>
                <blockquote>
                A vote for a referendum
                </blockquote>
                <table>
                    <tr>
                        <td><b>Name</b></td>
                        <td><b>Type</b></td>
                        <td><b>Description</b></td>
                    </tr>
                    <tr>
                        <td>ReferendumID</td>
                        <td>Array&lt;byte&gt;[32]</td>
                        <td>
                        The BLAKE2b-256 hash of the referendum to cast votes for
                        </td>
                    </tr>
                    <tr>
                        <td>Vote count</td>
                        <td>uint8</td>
                        <td>
                        How many votes are being cast. Must be equal to the amount of questions in the referendum
                        </td>
                    </tr>
                    <tr>
                        <td>Vote Answers</td>
                        <td>Array&lt;uint8&gt;[n]</td>
                        <td>
                        A list of answerIDs, one for each question.
                        First item is for question 1, second one for question 2, ...
                        </td>
                    </tr>
                  </table>
          </details>
      </td>
</table>

### Syntactic validation

If parsing of the data fails, the transaction is ignored for voting.

Validation of a singular referendum vote passes, if:

* The ID has been added to the node for voting
* The referendum is in progress (not in "Upcoming" or "Ended" state)
* The amount of votes cast is equal or smaller to the number of questions in the referendum

If a singular referendum vote fails, others are unaffected. For example, if one vote is for a referendum that the node is unaware of, others will still behave normally.

If there is no such answerID for a question, the question is ignored, and will cast no vote for this question only. As 0 never exists as an optionID, it can be used to skip a question but still vote for all others.

## Firefly Endpoints



## Node endpoints

<i>This is more of a quick overview. A more detailed version is in the works</i>

* GET /referendum/ : Lists all referenda, returning their UUID, the referendum name and status. Status `(upcoming,commencing,holding,ended)` must be used as a filter


* GET /referendum/{referendumID} : Gives a quick overview of the referendum. This does not include the questions or current standings.
* GET /referendum/{referendumID}/questions : Returns the entire vote with all questions, but not current standings.
* GET /referendum/{referendumID}/questions/{questionIdx} : Returns information and vote options for a specific question
* GET /referendum/{referendumID}/status : Returns the amount of tokens voting and the weight on each option of every question
* GET /referendum/{referendumID}/status/{questionIdx} : Return the amount of tokens voting for each option on the specified question

* POST /referendum/ : Submit a vote to track. By default not reachable from outside
* DELETE /referendum/{referendumID}: Removes a tracked vote.

# Drawbacks

* To vote, you always have to send tokens to yourself, so every time you receive them, you have to send them to yourself again if you want to include them in the vote. It is also technically possible to have different votes for different UTXOs on a single address.
* The standard referendum vote length would be 8 (Index) +2 (Header) +32 (ID) +1 (option count) +1 (option) = 44 additional bytes, which is rather long and might therefore increase the POW requirement for the transaction. However, additional questions on a referendum only take one extra byte.
* If someone has multiple addresses, they could be linked together if they cast a vote for all addresses simultaneously. Users will be made aware of this and have to do numerous votes with parts of their addresses in separated Firefly Wallets during the commencing period to reach the desired level of extra privacy.


# Unresolved questions

* Are the limits chosen reasonably?
* Do we need additional endpoints?
