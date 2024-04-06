# ecka-eg
[![ISC License](http://img.shields.io/badge/license-ISC-blue.svg)](https://github.com/pedroalbanese/ecka-eg/blob/master/LICENSE.md) 
[![GoDoc](https://godoc.org/github.com/pedroalbanese/ecka-eg?status.png)](http://godoc.org/github.com/pedroalbanese/ecka-eg)
[![Go Report Card](https://goreportcard.com/badge/github.com/pedroalbanese/ecka-eg)](https://goreportcard.com/report/github.com/pedroalbanese/ecka-eg)  

### ECKA-EG Key Agreement Protocol  

With verifiable encryption, Bob can prove to Alice that he has used a given encryption key of a ciphertext with a NIZK (Non-interactive Zero Knowledge Proof). In this case we will use ElGamal encryption and generate a proof of the public key which has been used for the encryption. If Bob uses Trent's public key to encrypt some ciphertext for Alice, then Bob can produce a proof that it has been encrypted with Trent's public key. Alice will then be able to check this against Trent's public key. 

<details>
  <summary>Theory</summary>    
	
We initially create a private key as a random number ![x](https://latex.codecogs.com/svg.latex?x) and a public key of:  

![Q = x \cdot G](https://latex.codecogs.com/svg.latex?Q%20%3D%20x%20%5Ccdot%20G)

With standard ElGamal encryption, we generate a random value ![r](https://latex.codecogs.com/svg.latex?r) to give:

![t = r \cdot Q](https://latex.codecogs.com/svg.latex?t%20%3D%20r%20%5Ccdot%20Q)

We then create a symmetric key from this elliptic curve point:

![AEADKey = \text{Derive}(t)](https://latex.codecogs.com/svg.latex?%5Ctext%7BAEADKey%7D%20%3D%20%5Ctext%7BDerive%7D%28t%29)

and where ![Derive](https://latex.codecogs.com/svg.latex?Derive) just converts a point on the curve to a byte array value that is the length of the required symmetric encryption key (such as for 32 bytes in the case of 256-bit Anubis).

Next, we compute the ciphertext values of:

![C1 = r \cdot G](https://latex.codecogs.com/svg.latex?C1%20%3D%20r%20%5Ccdot%20G)
![C2 = M \cdot H + r \cdot Q](https://latex.codecogs.com/svg.latex?C2%20%3D%20M%20%5Ccdot%20H%20%2B%20r%20%5Ccdot%20Q)

and where ![M](https://latex.codecogs.com/svg.latex?M) is the ![msg](https://latex.codecogs.com/svg.latex?%5Ctext%7Bmsg%7D) value converted into a scalar value. We then append these together to create the additional data that will be used for the symmetric key encryption of the message:

![AAD = C1 || C2](https://latex.codecogs.com/svg.latex?%5Ctext%7BAAD%7D%20%3D%20C1%20%7C%7C%20C2)

We then generate a nonce value (![Nonce](https://latex.codecogs.com/svg.latex?%5Ctext%7BNonce%7D)) and then perform symmetric key encryption on the message:

![cipher = \text{EncAEADKey}(\text{msg}, \text{Nonce}, \text{AAD})](https://latex.codecogs.com/svg.latex?%5Ctext%7Bcipher%7D%20%3D%20%5Ctext%7BEncAEADKey%7D%28%5Ctext%7Bmsg%7D%2C%20%5Ctext%7BNonce%7D%2C%20%5Ctext%7BAAD%7D%29)

The ciphertext then has values of ![C1](https://latex.codecogs.com/svg.latex?C1), ![C2](https://latex.codecogs.com/svg.latex?C2), ![Nonce](https://latex.codecogs.com/svg.latex?%5Ctext%7BNonce%7D), and ![cipher](https://latex.codecogs.com/svg.latex?%5Ctext%7Bcipher%7D). ![C1](https://latex.codecogs.com/svg.latex?C1), ![C2](https://latex.codecogs.com/svg.latex?C2) are points on the curve, and the ![Nonce](https://latex.codecogs.com/svg.latex?%5Ctext%7BNonce%7D) value and ![cipher](https://latex.codecogs.com/svg.latex?%5Ctext%7Bcipher%7D) are byte array values. To decrypt, we take the private key (![x](https://latex.codecogs.com/svg.latex?x)) and derive:

![t = x \cdot C1](https://latex.codecogs.com/svg.latex?t%20%3D%20x%20%5Ccdot%20C1)  
![AEADKey = \text{Derive}(t)](https://latex.codecogs.com/svg.latex?%5Ctext%7BAEADKey%7D%20%3D%20%5Ctext%7BDerive%7D%28t%29)  
![AAD = C1 || C2](https://latex.codecogs.com/svg.latex?%5Ctext%7BAAD%7D%20%3D%20C1%20%7C%7C%20C2)  
![msg = \text{DecAEADKey}(\text{cipher}, \text{Nonce}, \text{AAD})](https://latex.codecogs.com/svg.latex?%5Ctext%7Bmsg%7D%20%3D%20%5Ctext%7BDecAEADKey%7D%28%5Ctext%7Bcipher%7D%2C%20%5Ctext%7BNonce%7D%2C%20%5Ctext%7BAAD%7D%29)

Here is an overview of the method:

To generate the proof, we generate a random value (![r](https://latex.codecogs.com/svg.latex?r)) and a blinding factor (![b](https://latex.codecogs.com/svg.latex?b)) to give two points on the elliptic curve:

![R1 = r \cdot G](https://latex.codecogs.com/svg.latex?R1%20%3D%20r%20%5Ccdot%20G)
![R2 = r \cdot Q + b \cdot H](https://latex.codecogs.com/svg.latex?R2%20%3D%20r%20%5Ccdot%20Q%20%2B%20b%20%5Ccdot%20H)

Next, we create the challenge bytes with:

![chall = C1 || C2 || R1 || R2 || \text{Nonce}](https://latex.codecogs.com/svg.latex?%5Ctext%7Bchall%7D%20%3D%20C1%20%7C%7C%20C2%20%7C%7C%20R1%20%7C%7C%20R2%20%7C%7C%20%5Ctext%7BNonce%7D)

We take this value and hash it (![H()](https://latex.codecogs.com/svg.latex?H%28%29)), and create a scalar value with (![ek](https://latex.codecogs.com/svg.latex?ek)) to produce:

![c = H(\text{chall}) \cdot ek](https://latex.codecogs.com/svg.latex?c%20%3D%20H%28%5Ctext%7Bchall%7D%29%20%5Ccdot%20ek)

We then create two Schnorr proof values:

![S1 = b - c \cdot m](https://latex.codecogs.com/svg.latex?S1%20%3D%20b%20-%20c%20%5Ccdot%20m)
![S2 = r - c \cdot b](https://latex.codecogs.com/svg.latex?S2%20%3D%20r%20-%20c%20%5Ccdot%20b)

To verify the proof, we reconstruct ![R1](https://latex.codecogs.com/svg.latex?R1):

![R1 = c \cdot C1 + S2 \cdot G](https://latex.codecogs.com/svg.latex?R1%20%3D%20c%20%5Ccdot%20C1%20%2B%20S2%20%5Ccdot%20G)

We reconstruct ![R2](https://latex.codecogs.com/svg.latex?R2):

![R2 = c \cdot C2 + S1 \cdot Q + S1 \cdot H](https://latex.codecogs.com/svg.latex?R2%20%3D%20c%20%5Ccdot%20C2%20%2B%20S1%20%5Ccdot%20Q%20%2B%20S1%20%5Ccdot%20H)

This works because:

![\begin{align*} R2 & = c \cdot C2 + S1 \cdot Q + S1 \cdot H \\ & = c \cdot (b \cdot Q + m \cdot H) + (r - cb) \cdot Q + (b - cm) \cdot H \\ & = (cb + r - cb) \cdot Q + (cm + b - cm) \cdot H \\ & = r \cdot Q + b \cdot H \end{align*}](https://latex.codecogs.com/svg.latex?%5Cbegin%7Balign*%7D%20R2%20%26%20%3D%20c%20%5Ccdot%20C2%20%2B%20S1%20%5Ccdot%20Q%20%2B%20S1%20%5Ccdot%20H%20%5C%5C%20%26%20%3D%20c%20%5Ccdot%20%28b%20%5Ccdot%20Q%20%2B%20m%20%5Ccdot%20H%29%20%2B%20%28r%20-%20cb%29%20%5Ccdot%20Q%20%2B%20%28b%20-%20cm%29%20%5Ccdot%20H%20%5C%5C%20%26%20%3D%20%28cb%20%2B%20r%20-%20cb%29%20%5Ccdot%20Q%20%2B%20%28cm%20%2B%20b%20-%20cm%29%20%5Ccdot%20H%20%5C%5C%20%26%20%3D%20r%20%5Ccdot%20Q%20%2B%20b%20%5Ccdot%20H%20%5Cend%7Balign*%7D)

We then reconstruct the challenge with:

![chall = C1 || C2 || R1 || R2 || \text{Nonce}](https://latex.codecogs.com/svg.latex?%5Ctext%7Bchall%7D%20%3D%20C1%20%7C%7C%20C2%20%7C%7C%20R1%20%7C%7C%20R2%20%7C%7C%20%5Ctext%7BNonce%7D)

We take this value and hash it (![H()](https://latex.codecogs.com/svg.latex?H%28%29)), and create a scalar value with (![ek](https://latex.codecogs.com/svg.latex?ek)) to produce:

![c = H(\text{chall}) \cdot ek](https://latex.codecogs.com/svg.latex?c%20%3D%20H%28%5Ctext%7Bchall%7D%29%20%5Ccdot%20ek)

This value is then checked against the challenge in the proof, and if they are the same, the proof is verified.
</details>

### Usage  
 ```go
package main

import (
	"fmt"

	"os"

	"github.com/pedroalbanese/ecka-eg/core/curves"
	"github.com/pedroalbanese/ecka-eg/elgamal"
)

func main() {

	argCount := len(os.Args[1:])
	val := "hello"
	if argCount > 0 {
		val = os.Args[1]
	}

	domain := []byte("MyDomain")

	bls12381g1 := curves.BLS12381G1()
	ek, dk, _ := elgamal.NewKeys(bls12381g1)

	msgBytes := []byte(val)

	cs, proof, _ := ek.VerifiableEncrypt(msgBytes, &elgamal.EncryptParams{
		Domain:          domain,
		MessageIsHashed: true,
		GenProof:        true,
		ProofNonce:      domain,
	})

	fmt.Printf("=== ElGamal Verifiable Encryption ===\n")
	fmt.Printf("Input text: %s\n", val)
	fmt.Printf("=== Generating keys ===\n")
	res1, _ := ek.MarshalBinary()
	fmt.Printf("Public key %x\n", res1)
	res2, _ := dk.MarshalBinary()
	fmt.Printf("Private key %x\n", res2)
	fmt.Printf("=== Encrypting and Decrypting ===\n")
	res3, _ := cs.MarshalBinary()
	fmt.Printf("\nCiphertext: %x\n", res3)
	dbytes, _, _ := dk.VerifiableDecryptWithDomain(domain, cs)
	fmt.Printf("\nDecrypted: %s\n", dbytes)

	fmt.Printf("\n=== Checking proof===\n")
	rtn := ek.VerifyDomainEncryptProof(domain, cs, proof)
	if rtn == nil {
		fmt.Printf("Encryption has been verified\n")
	} else {
		fmt.Printf("Encryption has NOT been verified\n")
	}

	fmt.Printf("=== Now we will try with the wrong proof ===\n")
	ek2, _, _ := elgamal.NewKeys(bls12381g1)
	cs, proof2, _ := ek2.VerifiableEncrypt(msgBytes, &elgamal.EncryptParams{
		Domain:          domain,
		MessageIsHashed: true,
		GenProof:        true,
		ProofNonce:      domain,
	})

	rtn = ek.VerifyDomainEncryptProof(domain, cs, proof2)
	if rtn == nil {
		fmt.Printf("Encryption has been verified\n")
	} else {
		fmt.Printf("Encryption has NOT been verified\n")
	}

}
```

**Documentation**  
[BSI TR-03111 ECKA-EG (Elliptic Curve Key Agreement based on ElGamal)](https://www.bsi.bund.de/SharedDocs/Downloads/EN/BSI/Publications/TechGuidelines/TR03111/BSI-TR-03111_V-2-1_pdf.pdf?__blob=publicationFile&v=1)

## License

This project is licensed under the ISC License.

#### Copyright (c) 2020-2024 Pedro F. Albanese - ALBANESE Research Lab.
