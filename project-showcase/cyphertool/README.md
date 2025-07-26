
Cypher Tool (group task 1)

Team

	Taisto Valtteri Katainen 
	Henna Maria Helena Venho 
	Ahmed Seddiki

How this program works:

This command-line tool (program) greets the user, after which it will ask the user to...

	1. Choose the encryption operation (encrypt or decrypt)

	2. Choose the encryption type (ROT13 or Reverse or ROT8)

	3. Input the message to process (only 26 English alphabet letters are processed; non-alphabet characters are left unchanged)
	
After that the program will output the encrypted or decrypted message to the console.

How to run:

Inside of ./cyphertool local repository folder there has to be...

		cyphertool_main.go
		cyphertool_encryption.go
		cyphertool_input.go

After that, call the program from main.go with code:

		package main

		import "cyphertool"

		func main() {
			cyphertool.CypherTool()
		}

Call main.go from the linux terminal using "go run main.go"

Usage of the tool:

The Cypher Tool first greets the user, then it allows the user to select either 1 for encrypt, or 2 for decrypt as the operation.

If something else is chosen, the user is prompted to try again. 
Next the user is prompted to select the encryption type out of three options: 1 for Rot13, 2 forRreverse, or 3 Rot8. Again, if something else is chosen, the user is prompted to try again.

The user is then asked to enter the message to be encrypted or decrypted. 
The tool trims and validates the input, and then encrypts or decrypts the message with the chosen method and returns the outcome. Non-alphabet characters in the message are left unchanged.

Run example:

	Welcome to the Cypher Tool!
	
	Please select operation (write 1 or 2): 
	1. Encrypt
	2. Decrypt
	> 1
	
	Please select encryption type (write 1, 2 or 3): 
	1. ROT13
	2. Reverse
	3. ROT8
	> 2
	
	Please enter the message:
	> test message
	
	Encrypted message using Reverse:
	egassem tset
	
	Thank you for using the Cypher Tool program!

Cyphers used:

	1. Rot13 is a letter substitution cypher which replaces a letter with the 13th letter after it in the Latin alphabet. Therefore, 'a' becomes 'n', 'b' becomes 'o', and 'z' becomes 'm'.

	2. Reverse is a letter substitution cypher, where a letter is replaced with its reverse letter in the Latin alphabet, so that 'a' becomes 'z', 'y' becomes 'b', and so on.

	3. Rot8 is a letter substitution cypher which replaces a letter with the 8th letter after it in the Latin alphabet. Therefore, 'a' becomes 'i', 'b' becomes 'j', and 'z' becomes 'h'. 

This is the end of the readme file.