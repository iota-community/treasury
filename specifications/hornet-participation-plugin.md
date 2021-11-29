# RFC #06: HORNET Participation Plugin

- Feature name: `hornet-participation-plugin`
- Start date: 2021-11-24
- RFC PR: [iota-community/treasury#0006](https://github.com/iota-community/treasury/pull/6)

https://github.com/iota-community/treasury/pull/6

# Summary

This RFC defines a new HORNET plugin that enables token-holders to participate in On-Tangle voting or staking events. Specifically, it describes the structure and flow of a such an event, the format of participation data to be included in the transaction, and all endpoints the plugin provides.

# Motivation

[With the IOTA Foundation offering the community the opportunity to decide on the fortune of the unclaimed tokens](https://blog.iota.org/iota-community-treasury-and-genesis-validation), IOTA needs to provide a system for community members to create and vote on ballots. In community calls, the community figured out that the only way to achieve a fair ballot without investing years in R&D would be a simple token-based system. With this idea in mind, any token holders can use their tokens to vote for one of the options on each question, one token, one vote. During the weekly meetings, the majority of the community agreed to have further ongoing community ballots to establish a system and decide on the features of this system through voting. Therefore the system also needs to support future ballots, as well as multiple at the same time. Since the nature of staking is similar, the RFC was extended by this functionality.

The system describes below tries to address the following points:

* No protocol fork is required as voting data is stored in the data (indexation) payload of a value transaction
* Protection against flash loan attacks, where an attacker buys a lot of tokens for a short time to increase his influence in a ballot
  * This is achieved by adding a third factor to voting weight, which is holding time
* No token locking required
* Ballots can have multiple questions (allows compact voting without much overhead)
* Tokens remain in your wallet at all times


# Detailed design

## Event flow

An event has 4 different states

### Upcoming

Any event will start here. The creator specifies the starting times for each of the event states. Additionally, either a ballot with questions or the parameters to calculate staking rewards are included.

Events are neither automatically added to all nodes, nor are they broadcasted on the network. Instead, the event must be added to multiple nodes manually. Because submitting the same event to multiple nodes always results in the same ID, it is possible to compare the data of multiple nodes. This also provides security, as users can verify they are participating in the correct event.

### Commencing

Once the commence milestone defined in the event has been reached, the event enters the commencing phase. During this phase, users can participate in the event, but only the participation itself will be tracked by the nodes (i.e. votes will not be counted nor will staking rewards be distributed during this phase).

Whatever time is set as the beginning, there will always be a place in the world where it will be very inconvenient (e.g., the middle of the night). This phase grants fairness to every timezone, as it does not matter when you submit your participation in this phase. Also, it can be used to draw public attention and spread the word.

The node will ignore any participation for an event that is still upcoming or unknown. Users could still try to participate before the commencing phase, but the node will ignore those until the commence milestone is reached. This does not cause the validation of participations to fail. For example, if someone participates in three events at once but only one is known to the node, it will track the participation for the known one and ignore the other two.

### Holding

After the commencing phase is over, the holding phase will begin. During this phase the node will start counting the participation. This counting will happen first on the milestone following the `Start Milestone`, and the last time on the `End Milestone`. This phase is different between events with ballots and staking events.

#### Ballot events
Every time a milestone is confirmed, the node will first update the number of tokens voting for each option on every question (if there were changes in the ledger) and then add this amount to the total count for each option. The addition always adds weight for the following period since token balances cannot change until a new milestone is received. 

This adds a time factor to the ballot. If you buy tokens and vote during the ongoing event, you only get weight for the remaining time. Similarly, if you decide to sell, you will miss out on the weight for the remaining time. If you choose to change your opinion, your old opinion will get the weight of the elapsed time since the holding phase started, while your new opinion will get the weight of the remaining time.

Example: Assume Alice has 2i and does not vote at all. Now she sends her tokens to Bob 10000 milestones after the beginning of the holding period (approx. 1d 4h). Bob instantly votes for option A. After another 20000 milestones have passed, he sends the tokens to Charlie, who instantly votes option B. 5000 Milestones later, the event ends. Assuming the total supply consists of 2 iotas the outcome would be:
*     2i*10000=20000 are not counted in the voting
*     2i*20000=40000 for Build (80%)
*     2i* 5000=10000 for Burn (20%)
*     Turnout: 50000/70000=71.4% 

To avoid overflowing uint64, the vote power for each participation is calculated as the amount divided by 1000 rounded down. This means that every 1Ki represents 1 vote.

#### Staking events

Every time a milestone is confirmed, the node will increase the rewarded amount per address according to the staked amount and the configured calculation, i.e. by multiplying the staked amount using the numerator and denominator.

### Ended

Once the end milestone has been counted, the event is over. New participations are ignored, just as if the node did not know the event at all. 

## Event format

Users can submit an event to any nodes that have the endpoint open.

### Detailed format

<table>
    <tr>
        <th>Name</th>
        <th>Type</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>Event Name Length</td>
        <td>uint8</td>
        <td>
            The length of the Event name in bytes
        </td>
    </tr>
    <tr>
        <td>Event Name</td>
        <td>String</td>
        <td>
            The Event name in UTF-8 format
        </td>
    </tr>
    <tr>
        <td>Commence Milestone</td>
        <td>uint32</td>
        <td>
            The milestone where the event commences
        </td>
    </tr>
    <tr>
        <td>Start Milestone</td>
        <td>uint32</td>
        <td>
            The milestone where the event enters holding phase
        </td>
    </tr>
    <tr>
        <td>End Milestone</td>
        <td>uint32</td>
        <td>
            The milestone where the event ends
        </td>
    </tr>
    <tr>
        <td valign="top">Payload <code>oneOf</code></td>
        <td colspan="2">
            <details open="false">
            <summary>Ballot</summary>
            <blockquote>
                A ballot containing multiple questions.
            </blockquote>
            <table>
                <tr>
                    <th>Name</th>
                    <th>Type</th>
                    <th>Description</th>
                </tr>
                <tr>
                    <td>Payload Type</td>
                    <td>uint32</td>
                    <td>Set to <strong>value 0</strong> to denote a Ballot.</td>
                </tr>
                <tr>
                    <td>Questions Count</td>
                    <td>uint8</td>
                    <td>The count of questions proceeding</td>
                </tr>
                <tr>
                    <td valign="top">Questions <code>oneOf</code></td>
                    <td colspan="2">
                        <details open="false">
                        <summary>Question</summary>
                        <blockquote>
                            A question that can be voted on
                        </blockquote>
                        <table>
                            <tr>
                                <th>Name</th>
                                <th>Type</th>
                                <th>Description</th>
                            </tr>
                            <tr>
                                <td>Question Text Length</td>
                                <td>uint8</td>
                                <td>
                                    The length of the question in bytes
                                </td>
                            </tr>
                            <tr>
                                <td>Question Text</td>
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
                            <tr>
                                <td valign="top">Answer <code>oneOf</code></td>
                                <td colspan="2">
                                    <details open="true">
                                    <summary>Answer</summary>
                                    <blockquote>
                                        A possible answer for the question
                                    </blockquote>
                                    <table>
                                        <tr>
                                            <th>Name</th>
                                            <th>Type</th>
                                            <th>Description</th>
                                        </tr>
                                        <tr>
                                            <td>Answer Value</td>
                                            <td>uint8</td>
                                            <td>
                                                The value for this answer. Can be anything from 1-255. 0 and 255 are reserved.
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
                                                The length of the answer additional information text in bytes. Set to 0 if empty. Max 500 bytes are allowed.
                                            </td>
                                        </tr>
                                        <tr>
                                            <td>Answer Additional Information</td>
                                            <td>String</td>
                                            <td>
                                                Additional information about the Answer UTF-8 format. Field does not exist if the length was set to 0.
                                            </td>
                                        </tr>
                                    </table>
                                    </details>
                                </td>
                            </tr>
                            <tr>
                                <td>Question Additional Information Length</td>
                                <td>uint16</td>
                                <td>
                                    The length of the question additional information text in bytes. Set to 0 if empty. Max 500 bytes are allowed.
                                </td>
                            </tr>
                            <tr>
                                <td>Question Additional Information</td>
                                <td>String</td>
                                <td>
                                    Additional information about the question in UTF-8 format. Field does not exist if the length was set to 0.
                                </td>
                            </tr>
                        </table>
                        </details>
                    </td>
                </tr>
            </table>
            </details>
            <details open="false">
            <summary>Staking</summary>
            <blockquote>
                Information about staking rewards.
            </blockquote>
            <table>
                <tr>
                    <th>Name</th>
                    <th>Type</th>
                    <th>Description</th>
                </tr>
                <tr>
                    <td>Payload Type</td>
                    <td>uint32</td>
                    <td>Set to <strong>value 1</strong> to denote a Staking.</td>
                </tr>
                <tr>
                    <td>Description Text Length</td>
                    <td>uint8</td>
                    <td>
                        The length of the description text in bytes
                    </td>
                </tr>
                <tr>
                    <td>Description Text</td>
                    <td>String</td>
                    <td>
                        The description text of the staking in UTF-8 format
                    </td>
                </tr>
                <tr>
                    <td>Symbol Length</td>
                    <td>uint8</td>
                    <td>
                        The length of the token symbol in bytes. Must be between 3 and 10.
                    </td>
                </tr>
                <tr>
                    <td>Symbol Text</td>
                    <td>String</td>
                    <td>
                        The symbol in UTF-8 format
                    </td>
                </tr>
                <tr>
                    <td>Numerator</td>
                    <td>uint32</td>
                    <td>
                        The numerator used in the calculation of staking rewards. Must not be 0.
                    </td>
                </tr>
                <tr>
                    <td>Denominator</td>
                    <td>uint32</td>
                    <td>
                        The denominator used in the calculation of staking rewards. Must not be 0.
                    </td>
                </tr>
                <tr>
                    <td>Required Minimum Rewards</td>
                    <td>uint64</td>
                    <td>
                        The minimum rewards an address needs to reach to be counted in the final result.
                    </td>
                </tr>
                <tr>
                    <td>Additional Information Length</td>
                    <td>uint16</td>
                    <td>
                        The length of additional information in bytes. Set to 0 if empty. Max 500 bytes are allowed.
                    </td>
                </tr>
                <tr>
                    <td>Additional Information</td>
                    <td>String</td>
                    <td>
                        Additional information in UTF-8 format. Field does not exist if the length was set to 0
                    </td>
                </tr>
            </table>
            </details>
        </td>
    </tr>
    <tr>
        <td>Event Additional Information Length</td>
        <td>uint16</td>
        <td>
            The length of additional information in bytes. Set to 0 if empty. Max 2000 bytes are allowed.
        </td>
    </tr>
    <tr>
        <td>Event Additional Information</td>
        <td>String</td>
        <td>
            Additional information in UTF-8 format. Field does not exist if the length was set to 0
        </td>
    </tr>
</table>

_The BLAKE2b-256 hash of this object is the `EventID`, which is used as the UUID of the event_

### Syntactical validation

* The commence, start and end milestone need to be monotonic increasing
* The payload should not be empty
* A `Ballot` must contain between 1 and 10 questions

## Participation Format

Participation data will be sent via the indexation payload of a value transaction. To participate, you send a transaction to yourself, specifying your decisions.

### Criteria for a valid participation transaction

* Must be a valid value transaction adhering to protocol rules (e.g. dust protection)
* Has a singular output going to the same address as all input addresses.
  * Output Type 0 (SigLockedSingleOutput) and Type 1 (SigLockedDustAllowanceOutput) are both valid for this
* At least one input must come from the same address as the output. Multiple inputs are allowed. This is used to validate ownership of the address.
* The index must be ``"PARTICIPATE"``
* The data must be parseable

### Structure of the Participation

<table>
    <tr>
        <th>Name</th>
        <th>Type</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>Participations count</td>
        <td>uint8</td>
        <td>
        The amount of participations following. Must be at least one
        </td>
    </tr>
    <tr>
        <td valign="top">Participations <code>anyOf</code></td>
        <td colspan="2">
            <details open="true">
                <summary>Participation</summary>
                <blockquote>
                A participation for an event
                </blockquote>
                <table>
                    <tr>
                        <td><b>Name</b></td>
                        <td><b>Type</b></td>
                        <td><b>Description</b></td>
                    </tr>
                    <tr>
                        <td>EventID</td>
                        <td>Array&lt;byte&gt;[32]</td>
                        <td>
                        The BLAKE2b-256 hash of the event to participate in
                        </td>
                    </tr>
                    <tr>
                        <td>Answers count</td>
                        <td>uint8</td>
                        <td>
                        Must be equal to the amount of Questions in the Event Ballot. Can be 0 for non-Ballot events.
                        </td>
                    </tr>
                    <tr>
                        <td>Answers</td>
                        <td>Array&lt;uint8&gt;[n]</td>
                        <td>
                        A list of Answer Values, one for each question, in the same order as the Questions in the Ballot.
                        </td>
                    </tr>
                  </table>
          </details>
      </td>
</table>

### Syntactic validation

If parsing of the data fails, the participation is ignored.

Validation of the payload passes, if:
* At least 1 participation exists
* Each `EventID` is unique.


Validation of a singular participation passes, if:

* The ID has been added to the node for tracking
* The event is in progress (not in "upcoming" or "ended" state)
* The amount of Answers is equal to the number of `Questions` in the event `Ballot` or 0 for `Staking` events.

If a singular participation is invalid, others are unaffected. For example, if one participation is for an event that the node is unaware of, others will still behave normally.

If there is no such Answer Value for a Question, the participation for this Question will be tracked as invalid with the reserved value `255`.
As `0` never exists as a valid `Answer Value`, it can be used to skip a `Question` but still answer all others.

## Public Node endpoints

* GET `/api/plugins/participation/events`: Lists all events, returning their EventID. Optional `type` query parameter to filter by payload type.
* GET `/api/plugins/participation/events/{eventID}`: Returns the event information as a JSON payload.
* GET `/api/plugins/participation/events/{eventID}/status`: Returns the status of the given event `(upcoming,commencing,holding,ended)` and if it contains a `Ballot`, the current and accumulated answers for each question. For `Staking` events it contains the amount of funds currently staking and the amount of tokens rewarded.
* GET `/api/plugins/participation/outputs/{outputID}`: Returns the events the given output is participating in or participated in the past if the output was spent.
* GET `/api/plugins/participation/addresses/{bech32Address}`: Returns the staking events the address participated in and the amount of rewards received.
* GET `/api/plugins/participation/addresses/ed25519/{ed25519Address}`: Returns the staking events the address participated in and the amount of rewards received.

## Private Node endpoints

This node endpoints should be kept private and only be used by the node operators.

* POST `/api/plugins/participation/admin/events`: Can be used by the node operator to add a new event to the node for tracking. Events that already started or event already ended can be added, as long as the node has enough history to calculate the participation.
* DELETE `/api/plugins/participation/admin/events/{eventID}`: Can be used by the node operator to remove an event that is being tracked and all the data corresponding to that event.
* GET `/api/plugins/participation/admin/events/{eventID}/active`: Can be used by the node operator to list all outputs actively participating for the given event.
* GET `/api/plugins/participation/admin/events/{eventID}/past`: Can be used by the node operator to list all outputs that participated for the given event and are not currently active.
* GET `/api/plugins/participation/admin/events/{eventID}/rewards`: Can be used by the node operator to list all rewards calculated for the given staking event. This endpoint will filter out any addresses not reaching the `Required Minimum Rewards` of the event. This also means that this endpoint will return the corrected total rewarded amount for the event.

