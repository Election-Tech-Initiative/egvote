# Creating a Verifiable Election

ElectionGuard is a software development kit (SDK) election system vendors can use to implement end-to-end verifiable (E2E-V) elections.  Because it is designed to be integrated into both existing and new systems, it needs to be flexible. However, for end-to-end verifiability to be achieved, the following 3 core capabilities must be delivered:

* The voter is given a ```verification code``` they can use to verify their ballot was included in the final tally
* The voter has the means to ```challenge a ballot```
* The election results and all its encrypted artifacts, including zero knowledge proofs, are published to enable ```full independent verifiability``` by third parties
  
We describe each of these capabilities in detail below.

## Voter Verification Code

A voter verification code is generated by the ballot encryption process of ElectionGuard. It is generated when the user has ```submitted their ballot``` to the voting system.  

### Generating the Verification Code

Submitting a ballot can mean virtually pressing a "Submit" button on a touchscreen voting device, inserting a paper ballot into a digital scanner, or some other event that causes a [cast vote record][nist-cast-vote-records] to be generated in the voting system.

From a user experience standpoint, the ballot encryption occurs at a point after the voter has filled out all contests they plan to, and has had a chance to review all their choices in a summary screen (or the completed paper ballot).  In the ballot marking device use case, the ballot would be encrypted after the voter presses submit.

![Schematic of Ballot Marking Device using ElectionGuard][schematic-ballot-marking-device]

For a ballot scanner, the cast vote record is generated after the scanner has interpreted the ballot.

![Schematic of Scanner-based system using ElectionGuard][schematic-ballot-scanner-device]

> *Please note that while the diagrams above show devices such as 'encrypted ballot server', __ElectionGuard is intended to be run in an offline environment__; as such, if an API or server is used to listen for or communicate encryption requests, they are expected to be run locally in a secure environment.*

Once the ballot has been encrypted, the encrypting device needs to handle the verification code returned by the ballot encryption process and present it to the voter, ideally by printing it at the moment of creation in a format easy for the voter to take with them.  When the election results are published (see "Publishing election results" below), these verification codes are published alongside, enabling them to see that their ballots were included in the election tally (or, in the case of Challenge Ballots, that they were included as challenge ballots (see "Challenging a Ballot" below))

[![Sample ElectionGuard verification code output by tracking site][sample-election-guard-verification-code-image]][election-guard-verification-code-demo]

The verification code format is a mix of human-readable words and alphanumeric codes. The first word is always human-readable to facilitate a search-like discovery experience (to try, go to [our demo tracking site][election-guard-demo-site] and begin typing 'coo' to surface the verification code above, or simply click the image to be taken directly).

## Challenging a ballot

Encrypting a ballot and generating a verification code is the first step in an E2E-V process for the voter. For E2E-V compliance, the voter has to be offered the ability to ```challenge``` or "spoil" the ballot after the verification code has been generated and provided to the voter.

When a ballot is challenged, it is no longer eligible to be included in the final tally. If the voter wants to register a vote after a challenge, they will need to begin the process again with a new ballot.

In addition, when a ballot is challenged, the system must be able to perform a decryption of the challenged ballot and reveal its contents to the voter.

Challenging of ballots increases the inherent security of the system, since the system has to assume that any ballot encrypted might have to be revealed, exposing any manipulation that may have occurred after submission; the system can't assume its actions will remain hidden. Individual voters can obtain as much confidence as they desire by challenging as many ballots as they wish.  Although they won't ever be shown that the encryption of a cast ballot matches their selections, voters can see that every challenged ballot was correctly encrypted and committed to before the decision to cast or challenge was made. The ability to observe that all challenge ballots are correct thereby provides indirect evidence that cast ballots were also correct.

## Publishing Verifiable results

When the election is complete and the results tallied, E2E-V elections publish the results so they can be verified independently.

E2E-V election verifiers are written by third parties according to their interpretation of the [ElectionGuard Specification][election-guard-specification]. Any data published by an election should enable each and every cast encrypted ballot to be interrogated and tallied independently. From the spec:

> An ElectionGuard ballot is comprised entirely of encryptions of one (indicating selection made) and zero (indicating selection not made). Two things must be proven about the encryption of each vote:
>
1. The encryption associated with each option is either an encryption of zero or an encryption of one.
2. The sum of all encrypted values in a contest is equal to the selection limit for that contest (usually one).

> [ElGamal encryption][elgamal-encryption] enables efficient zero-knowledge proofs of these requirements, and the [Fiat-Shamir heuristic][fiat-shamir-heuristic] can be used to make these proofs non-interactive.  Chaum-Pedersen proofs are used to demonstrate that an encryption is that of a specified value, and these are combined with the [Cramer-Damgård-Schoenmakers technique][proofs-partial-knowledge-witness-hiding] to show that an encryption is that of one of a specified set of values – particularly that a value is an encryption of either zero or one.  The set of encryptions of selections in a contest are homomorphically combined, and the result is shown to be an encryption of that contest’s selection limit, again using a Chaum-Pedersen proof.

Once every ballot is proven to be properly formed (as above), all of the votes for each option are homomorphically summed to produce encryptions of the tallies for each option.  The final step is then to decrypt these tallies and provide additional proofs that the decryption are correct.

### Showing Verification Code Results to Voters

In addition to satisfying the data requirements for proper verification, any publishing exercise needs to also enable individual voters to query for the ballot that matches their verification code.

Verification code results of cast ballots will indicate that the ballot was included in the final tally, and any associated metadata about the device that recorded the ballot (time, location, election, etc.; see example above).

Challenge ballots are also published. Results for verification codes of challenge ballots should indicate that the ballot was ***not*** included in the election tally. Because the ballot is not included in the election tally, the contents can be decrypted and presented as well.

<!-- Links -->
[nist-cast-vote-records]: https://github.com/usnistgov/CastVoteRecords "NIST Cast Vote Records Github"
[sample-election-guard-verification-code-image]: https://res.cloudinary.com/electionguard/image/upload/v1596647319/verification-code_k82e8f.jpg
[election-guard-verification-code-demo]: https://demo.electionguard.vote/track/cook%207HMCG%20notion%209329D%20bandwidth%2099DCF%20mist%207M792%20panpipe%20BF7C9%20corsage%204CMGC%20privilege%2044J47%20daybed%20GBH74 "Election Guard verification demo"
[election-guard-specification]: https://raw.githubusercontent.com/wiki/microsoft/electionguard/Informal/ElectionGuardSpecificationV0.85.pdf "Election Guard Specification - Microsoft Research"
[schematic-ballot-marking-device]: https://res.cloudinary.com/electionguard/image/upload/v1596565406/eg-bmd-integration-2_qnar1h.png "Schematic of ballot marking device Using Election Guard"
[schematic-ballot-scanner-device]: https://res.cloudinary.com/electionguard/image/upload/v1596565608/eg-scanner-integration_zprscw.png "Schematic of ballot scanning device Using Election Guard"
[elgamal-encryption]: https://en.wikipedia.org/wiki/ElGamal_encryption "ElGamal encryption"
[fiat-shamir-heuristic]: https://en.wikipedia.org/wiki/Fiat%E2%80%93Shamir_heuristic "Fiat-Shamir Heuristic"
[proofs-partial-knowledge-witness-hiding]: https://www.win.tue.nl/~berry/papers/crypto94.pdf "Proofs of Partial Knowledge and Simplified Design of Witness Hiding Protocols"
[election-guard-demo-site]: https://demo.electionguard.vote "Election Guard demo tracking site"