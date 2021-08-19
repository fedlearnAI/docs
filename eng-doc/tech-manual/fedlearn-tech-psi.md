JDT Fedlearn Platform Technical Manual
============

Private-Preserving Entity Match
Private-Preserving Entity Match, alias ID Matching, can be also called Secure Entity Alignment, Secure Entity Resolution. It is a cryptographic protocol that allows two or more parties to input their respective private sets and obtain their intersection set under private-preserving methods.

Our platform now supports four methods for private-preserving entity match: MD5, RSA, Diffie-Hellman and Freedman.

#### 3.2.1 ID Matching With MD5
ID matching with MD5 (Message-Digest algorithm 5) supports two-party or multi-party ID matching. It mainly involves three phases.

>1：Requests Phase
> 
> Coordinator send requests to each client for starting ID matching. This phase does not involve data communication.
> 
>2：Encrypt private ID Phase
> 
> Each client receives request from Coordinator and generate an MD5 hash from each of the private ID. All of the MD5 hash are then transferred Coordinator. The original IDs are not revealed to any other clients or Coordinator because of the irreversible property of MD5 hash.
>  
>3: Matching Phase
> 
> Coordinator collects each encrypted private ID set and calculates for the intersection. The intersection is transferred to each client after the calculation.
> 
> **According to the above procedure, all private IDs are hashed with MD5 which is irreversible. Therefore, there is no data breach using ID matching with MD5 **

Advantages: ID Matching with MD5 is efficient, fast-speed and simple-procedure. 

#### 3.2.2 ID Matching with RSA algorithm

> ID Matching with RSA algorithm relies on the encryption and the co-prime properties. It has six phases and only support two-party ID Matching for now.
> 1: Coordinator construct two universal hashing function H 和 H' and sends requests for alignment to each client including H and H'. This phase does not include encrypted data communication.
> 
> 2: Each of the two parties need alignment generate hash for each of the local IDs with the first universal hashing function H, noted 
> H<sub>id<sub>1</sub></sub>, H<sub>id<sub>2</sub></sub>.
> One of the two parties generate a pair of the public key (n, e) and private key (n, d), and sends the public key to the coordinator.
> 
> 3: Coordinator receives the public key and sends it to the other client. Upon reception, the other receive the public key and generate a random number for each of the local IDs, noted R<sub>id</sub>, for protection.
> Apply RSA encrpytion to R<sub>id</sub> and multiply H<sub>id<sub>2</sub></sub> with (R<sub>id</sub>)<sup>e</sup> mod n respecitively and obtain y = (H<sub>id</sub> * (R<sub>id</sub>)<sup>e</sup>) mod n which is later sent to the coordinator.
> 
> 4: Coordinator receives the encrypted result and sends it to the first client which receives the result and sign it to get y' = (y<sup>d</sup>) mod n.
> Meanwhile, the first client sign its local hash from phase 2 and get a second layer universal hash with H' using t<sub>1</sub> = H'((H<sub>id<sub>1</sub></sub>)<sup>d</sup> mod n) which is later sent to the coordinator.
>
> 5: Upon reception of y' and t<sub>1</sub>, Coordinator sends them to the second client which removes the random number R<sub>id</sub> from y',
> and get a second layer of universal hash with t<sub>2</sub> = H'((H<sub>id<sub>2</sub></sub>)<sup>d</sup> mod n).
> At this time, the second client has both of the two-layer hashing result. It calculate the intersection and send the signed intersection to the coordinator.
>
> 6: Coordinator collects the alignment result and sends back to the first client.
>
> **In the above procedure, coordinator plays a role of encrypted data distributor. Therefore, neither of the clients reveals the original IDs, proving no data breach.**
>
> **The above procesure shows a two-party ID Matching with RSA algorithm. It will be extended to multiple parties in the near future. **

Advantages: ID Matching with RSA Algorithm can be of advantages when the two parties need alignment have a huge difference in the number of local IDs. The communication cost can be saved under this circumstance.

#### 3.2.3 ID Matching with Diffie-Hellman Algorithm

ID Matching with Diffie-Hellman Algorithm is based on a key-exchange protocol whose core idea enables mutual secret between two parties. Two successive encryptions get the same result regardless the order, noted Enc<sub>A</sub>(Enc<sub>B</sub>(ID)) = Enc<sub>B</sub>(Enc<sub>A</sub>(ID)). This ID Matching method supports for two-party or multi-party ID Matching requests.

ID Matching with Diffie-Hellman Algorithm has four phases.
> 
> 1：Initialization Phase
> 
> In the initialization phase, coordinator generates two global parameters g and n for future encryption at each client. The coordinator randomly selects a client to be the active party and sends g and n to it (All the other parties are then noted as passive parties). This phase does not involve any encrypted data communication.
> Each client generate a random number for future encryption. Upon reception of g and n, The active party generate encryption Enc<sub>A</sub>(ID<sub>A</sub>) for its local IDs and sends its encryption of local IDs to the coordinator.
> 
> 2：Double Encryption for IDs from the Active Party Phase
> 
> Coordinator receives the encrypted IDs from the active party (Enc<sub>A</sub>(ID<sub>A</sub>)) and sends Enc<sub>A</sub>(ID<sub>A</sub>) with g and n to all the passive parties,
> All the passive parties first encrypts Enc<sub>A</sub>(ID<sub>A</sub>) after receiving it and generate Enc<sub>P<sub>i</sub></sub>(Enc<sub>A</sub>(ID<sub>A</sub>))), then it encrypts each of the local IDs for the first time and get Enc<sub>P<sub>i</sub></sub>(ID<sub>P<sub>i</sub></sub>) respective. Both of them are sent to the coordinator at the end of this phase.
> 
> 3：Double Encryption for Passive-Party IDs and Active-Party IDs Storage Phase
> 
> Coordinator receives Enc<sub>P<sub>i</sub></sub>(Enc<sub>A</sub>(ID<sub>A</sub>)) and Enc<sub>P<sub>i</sub></sub>(ID<sub>A</sub>) from each passive parties and sends all the result to the active party.
> The active party double encrypts each of the passive-party IDs and get Enc<sub>A</sub>(Enc<sub>P<sub>i</sub></sub>(ID<sub>P<sub>i</sub></sub>)). For each of the passive party i, get an alignment result with Enc<sub>P<sub>i</sub></sub>(Enc<sub>A</sub>(ID<sub>A</sub>))) and Enc<sub>A</sub>(Enc<sub>P<sub>i</sub></sub>(ID<sub>P<sub>i</sub></sub>)). Finally, it calculates the final alignment result with respect to all the sub-alignment results. The active party stores the final intersection result at this phase and sends the aligned double encrypted IDs and the double encrypted IDs for each of the passive parties respectively to the coordinator.
> 
> 4：Passive-Party IDs Storage Phase 
> 
> Coordinator collects all the double encryted IDs and sends the double ciphers to each of the passive parties respectively.
> Each passive party then get the decrypted intersection result and stores it locally.

Advantages：ID Matching with Diffie-Hellman Algorithm supports multi-party alignment with only double encryption for each of the party with privacy preserved, indicating both efficiency and safety.

#### 3.2.4 ID Matching with Freedman Algorithm

ID Matching with Freedman Algorithm is based on additive homomorphic encryption of parameters of a polynomial. It constructs a polynomial function given the roots to be of the parties IDs. Only the intersection enables the polynomial function to be 0. This methods supports two-party or multi-party ID matching requests.

**Only supports small digit IDs. Float allowed.**

ID Matching with Freedman Algorithm has five phases.

> 1：Initialization Phase
> 
> Coordinator sends requests for each of the client, asking for length of the private IDs. Each client masks its length with a small random number and sends the masked value to the coordinator.
> 
> 2：Active Party Solves for The Parameters Phase
> 
> Coordinator chooses the party with the minimum length to be the active party after receiving the masked lengths, and it sends a request to the active party to solve for the polynomial f(x) = ∑ß<sub>u</sub>x<sup>u</sup>.
> The active party solves for the polynomial parameters with lagrange interpolation and gets ß<sub>0</sub>, ß<sub>1</sub>, ..., ß<sub>n</sub>. Each of the parameter is encrypted with Paillier and the encrypted parameter is sent back to the coordinator with the public key.
> 
> 3：Passive Party Calculation Phase
> 
> Coordinator collects the encrypted parameters for the polynomial along with the public key and sends all to the passive parties.
> Each passive party generates a random number r and calculates r * f(y<sub>i</sub>) + y<sub>i</sub> for each of the local ID y<sub>i</sub>. The results are sent back to the coordinator from each passive party.
> 
> 4：Active Party Alignment and Result Storage Phase
> 
> Coordinator collects polynomial results form each passive party (f(y<sub>i</sub>)) and send them to the active party. 
> The active party decrypts all the results and do the alignment. The alignment result is saved locally and the aligned index for each passive party is sent to the coordinator to be sent back to each passive party.
>
> 5：Passive Party Result Storage Phase
> 
> Coordinator sends the aligned index to each passive party respectively. Upon reception, each passive party saves the original IDs with the aligned index locally.


















