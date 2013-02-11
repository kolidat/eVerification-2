<a href="http://crises-deim.urv.cat/everification2/" target="_blank"><img src="https://raw.github.com/CRISES-URV/eVerification-2/master/figures/logoeverification2.png" />

eVeriﬁcation2 TSI-020100-2011-39 is a research project leaded by Scytl Secure Electronic Voting S.A.,
with the collaboration of CRISES research group from Universitat Rovira i Virgili, and supported by 
the Spanish Ministry of Industry, Commerce and Tourism (through the development program AVANZA I+D).

<a href="https://www.planavanza.es" target="_blank"><img src="https://raw.github.com/CRISES-URV/eVerification-2/master/figures/logo_planAvanza2.png"  width="300" height="150">

<center><table border="0">
<tr><td><a href="http://www.scytl.es" target="_blank"><img src=https://raw.github.com/CRISES-URV/eVerification-2/master/figures/logoScytl.png border="0"></td>
<td><a href="http://www.urv.cat" target="_blank"><img src=https://raw.github.com/CRISES-URV/eVerification-2/master/figures/logoURV.png border="0"></td>
<td><a href="http://crises-deim.urv.cat" target="_blank"><img src=https://raw.github.com/CRISES-URV/eVerification-2/master/figures/logoCrises.png width="140" height="50" border="0"></td></tr>
</table></center>

You can find more information about the eVerification2 project in http://crises-deim.urv.cat/everification2

#TTP SmartCard-Based ElGamal Cryptosystem Using Threshold Scheme for Electronic Elections

As a result of this research project, CRISES group has studied the feasibility of developing ElGamal 
cryptosystem and Shamir’s secret sharing scheme into JavaCards, whose API gives no support for it.

In particular, the contributions of our work have been the design and development of
the following building blocks for JavaCards: (i) ElGamal cryptosystem to generate the ElGamal key pair, (ii) Shamir’s 
secret sharing scheme to divide the private key in a set of shares, (iii) secure communication channels 
for the distribution of the shares, and (iv) a decryption function without reconstructing the private key. 
This solution can be useful for a typical e-voting system, speciﬁcally in the voting scheme presented by 
Cramer et al. [<a href="#ref1">1</a>].

You can find more information about these contributions and how they were designed and implemented in the 
conference <a href="https://raw.github.com/CRISES-URV/eVerification-2/master/paper.pdf">paper.pdf</a> presented in Foundations & Practice of Security 2011 called: TTP SmartCard-Based ElGamal 
Cryptosystem Using Threshold Scheme for Electronic Elections [<a href="#ref2">2</a>]. In the <a href="https://raw.github.com/CRISES-URV/eVerification-2/master/extendedpaper.pdf">extendedpaper.pdf</a>, you can find a 
description of an execution example.


##Software

This library implements the protocol described in the paper mentioned above and it is prepared to easily execute a 
configurable example with a maximum number of shares (n=5) and a threshold from 2 to the maximum of shares (n).
To execute the following functions implemented in the library:
- <b>Generate </b> a set of shares from a SmartCard(SC) according to user configuration (number of shares, threshold and key size).
- <b>Distribute </b> the generated shares, public key, and other public parameters from that SC to the rest of SCs.
- <b>Verify </b> the received share from each SC. 
- <b>Encrypt </b> a value using the public ElGamal parameters.
- <b>Partial decrypt </b> from each SC.
- <b>Homomorphic Recount </b> (in a voting context) through the aggregation of the set of partial decryptions.

The code is divided into two different parts: thresholdClient and thresholdLib code.
The former part includes the GUI and the code related to manage of the protocol execution as a client. This part has been developed
in Java programming language.
The last part is the applet code placed/installed into each SC, which is written in JavaCard.

Once the applet has been installed in each SC, our application, following the protocol described in the paper, initiates the APDU communication, as detailled below:

### Shares generation process </b>(shares are generated by any SC, then this SC is called generator SC).
		
		1. The Applet has been selected and initialized. In this case, the applet identifier is 3132333433123450. The key size of ElGamal cryptosystem is fixed here. The sent APDUs are as follows:
			- Applet selection: CLA=00, INS=a4, P1=04, P2=00, Lc=08, DATA=aID=3132333433123450, Le=00.
			- Applet initialization: CLA=80, INS=00, P1=0x, P2=00 (where x is the key size, it can be 2=512, 3=768, 4=1024, 5=1280, 6=1536, 7=1984 and 8=2048 bits).
		
		2. In order to generate the following values, which are requiered by ElGamal cryptosystem initialization,into the SC, the following APDUs are sent: 
			- p: CLA=90, INS=01, P1-P2=00, Lc=20 and data=p. 
			- q: CLA=90, INS=11, P1-P2=00, Lc=20 and data=q. 
			- g: CLA=90, INS=02, P1-P2=00, Lc=20 and data=g. 

		3. In order to securely build the private and public key, which are requiered by ElGamal cryptosystem, into the SC, the APDUs are as follows:
			- Pk: CLA=80, INS=03, P1-P2=00.
			- Sk: CLA=80, INS=04, P1-P2=00.
			- The SC generator takes the id=0, this is stored by APDU: CLA=80, INS=12, P1-P2=00, Lc=1, DATA=id, Le=00.

		4. Once the private key is generated, it must be split in some shares. The following APDUs fix the shares and threshold number, and their generation:
			- Threshold and shares number storing: CLA=80, INS=1e, P1=2-5,P2=3-5 (where P1 and P2 are the threshold and the shares number respectively).
			- (t-1) coefficients generation: CLA=80, INS=05, P1-P2=00.
			- (t-1) coefficient commitments generation: CLA=80, INS=06, P1-P2=00.
			- Evaluation values generation: CLA=80, INS=07, P1-P2=00.
			- (N) shares generation: CLA=80, INS=08, P1-P2=00.
			- (N) shares commitments generation: CLA=80, INS=09, P1-P2=00.

		5. The client gets, from the SC, the public parameters generated by threshold scheme, coefficient commitment and Pk:
			- Coefficient commitment getting: CLA=80, INS=1B, P1-P2=00.
			- Pk getting: CLA=80, INS=1A, P1-P2=00.

		6. Finally, the SC must verify the correctness generation of his own share and share commitment (remember that generator SC may also participate in the secret reconstruction step):
			- Share verification: CLA=80, INS=A0, P1-P2=00.
			- Share commitment verification: CLA=80, INS=B0, P1-P2=00.


<img src="https://raw.github.com/CRISES-URV/eVerification-2/master/figures/generation.png">

    
### Share Distribution between generator SC and receiver SC (this step must be repeated in each receiver SC).

		Steps 1 and 2 from shares generation process are repeated here.

		3. The public parameters of threshold scheme, generated by generator SC for each different receiver SC, are recovered by sending the following APDUs to generator SC:			
			- Share getting: CLA=80, INS=19, P1= smart card identifier,P2=00.
			- Share commitment getting: CLA=80, INS=1C, P1= smart card identifier, P2=00.
			Note that SC identifier must be different in each receiver SC since there is no equal share.

		4. The threshold parameters generated in the generation step, including the corresponding share and share commitment, are stored in the receiver SC:
			- Threshold and shares number storing: the APDU is: CLA=80, INS=1e, P1=2-5,P2=3-5
			- Coefficient commitment storig: CLA=90, INS=16, P1-P2=00.
			- Card identifier storing: CLA=80, INS=12, P1-P2=00, Lc=01 and data=id_card.
			- (own)Share storing: CLA=90, INS=14, P1-P2=00, Lc=20 and data=share.
			- (own)Share commitment storing: CLA=90, INS=17, P1-P2=00, Lc=20 and data=share commitment.
			- Public key storing: CLA=90, INS=15, P1-P2=00, Lc=20 and data=public key.
			- Evaluation values generation: CLA=80, INS=07, P1-P2=00.

		5. Once the receiver SC has stored their own share and share commitment, it must verify the correctness generation made by generator SC.
			- Share verification: CLA=80, INS=A0, P1-P2=00.
			- Share commitment verification: CLA=80, INS=B0, P1-P2=00.


<img src="https://raw.github.com/CRISES-URV/eVerification-2/master/figures/distrib.png">


### Verification (any SC with stored share and share commitment can perform this step)

		Step 1 from shares generation process are repeated here.
		2. Once the receiver SC has stored their own share and share commitment, it must verify the correctness generation made by generator SC.
			- Share verification: CLA=80, INS=A0, P1-P2=00.
			- Share commitment verification: CLA=80, INS=B0, P1-P2=00.

### Encryption (any SC can perform this step)

		Steps 1 and 2 (if they are not performed before) from shares generation process are repeated here.
		3. The data to encrypt is sent to SC and it is encrypted with APDU: CLA=90, INS=0C, P1-P2=00, Lc=20, Data=message to encrypt.
		4. ElGamal encryption result is recovered by sending the APDU: CLA=90, INS=0D, P1-P2=00.


### Homomorphic Decryption or tally (as least t-SC must participate in this process)
		Once elections have been concluded, a set of at least t-members of the electoral board is necessary to meet successfully decrypting votes.
		At this point, votes have been aggregated yet thanks to the use of a homomorphic properties of ElGamal.
		
		-By using the lagrange coefficients, the evaluations values, and the aggregated votes computed before in the client side, each of t-SCs compute securely and internally its partials Y1:
			Step 1 from shares generation process is repeated here.
			2. Lagrange coefficient sending: CLA=90, INS=13, P1= card identifier, P2=00, Lc=20 and data=Lagrange coefficient.
			3. Data to decrypt sending: CLA=90, INS=1D, P1-P2=00, Lc=20 and data=Y2+Y1.
			4. Partial decryption: CLA=80, INS=0F , P1-P2=00.
		
		Once all partial decryptions have been recovered, in the client side the final decryption is computed by following the steps described in the paper.
		
		
<img src="https://raw.github.com/CRISES-URV/eVerification-2/master/figures/tally.png">


You can find more information about the implementation in the section Development Details of the <a href="https://raw.github.com/CRISES-URV/eVerification-2/master/extendedpaper.pdf">extendedpaper.pdf</a>


##License

This software is released under BSD 3-clause license which is contained in the file <a href="https://github.com/CRISES-URV/eVerification-2/blob/master/LICENSE">LICENSE</a>.


##Future Work

As future work, we are working on a non-trusted third party (Non-TTP)
solution with a distributed generation of the shares. In addition, we would like
to improve the efficiency, time and storage of the protocol in smartcards (i.e.,
using ElGamal on elliptic curves).


#Bibliography

<a name="ref1"></a>[1] Cramer, R., Gennaro, R., Schoenmakers, B.: A secure and optimally ecient
multi-authority election scheme. In: Proceedings of the 16th annual international
conference on Theory and application of cryptographic techniques. pp. 103{118.
EUROCRYPT'97, Springer-Verlag, Berlin, Heidelberg (1997), 
http://portal.acm.org/citation.cfm?id=1754542.1754554

<a name="ref2"></a>[2] J. Pujol-Ahullo, R. Jardi-Cedo, J. Castella-Roca, O. Farràs , 
"TTP SmartCard - based ElGamal Cryptosystem using Threshold Scheme for Electronic Elections ", 
Foundations & Practice of Security 2011 - FPS 2011, Paris, France, May 2011. 
http://crises2-deim.urv.cat/docs/publications/conferences/656.pdf

