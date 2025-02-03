# Art Decoder/Encoder

By Henna Venho
- 2nd task at kood/Sisu (April 2024)

## Introduction 

**Art Decoder/Encoder** is a versatile application that not only decodes art data into text-based ASCII art from the command line but also provides a web interface for user-friendly interaction. This innovative tool offers the capability to transform single-line or multi-line art data into beautifully rendered text-based art, or to encode text-images into an encoded format.


## Technologies Used: 

- Golang Standard Library
- HTML
- CSS
- Visual Studio Code
- WSL (Ubuntu)
- Version Control System (Git)
- Repository (Gitea)
- VCS Client (Bash)
- Resources from ASCII Art Archive: https://www.asciiart.eu/
- Several Browsers for Testing


## Features

* **Command-Line Interface**: For traditional CLI enthusiasts, directly convert strings into art from your terminal.
* **Web Interface**: A modern web interface for easy encoding and decoding with just a few clicks.
* **Multi-Line Support**: Handles multi-line input and output with ease.
* **Encode/Decode Toggle**: Seamlessly switch between encoding and decoding modes.


## Getting Started

**Prerequisites**
* Go (Version 1.21.6 or later)
* A modern web browser

**Installation**
1. Clone the repository to your local machine:

> `$ git clone https://github.com/HennaVenho/art.git`

2. Navigate to the cloned directory:

> `$ cd art`


**Building the Tool**

To build the project, use the following command (optional):

> `$ go build -o art .`

This command compiles the application and generates an executable named 'art'.

If you built the program, you can run it with `./art <arguments>`; otherwise you can run it with `go run . <arguments>`.


**Running the Server**

Start the web server using the compiled executable:

> `$ ./art -s`

Upon execution, the server will start listening on 'localhost:8080'. You can access the web interface by opening your web browser and visiting:

> `http://localhost:8080`


## Web Interface Usage

* **Decode**: Enter the encoded string into the text area and click the "Decode" button to view the text-based art.
* **Encode**: To encode your art, enter your art into the text area, and click the "Encode" button.
* **Clear**: Clears the current input and output fields.


## Command-Line Usage: 

For those who prefer the command-line interface, here's how to interact with the Art Decoder/Encoder:

* **Decoding**: 

Use the string to be decoded as the argument, inside single or double quotation marks: 

> `$ ./art '[5 #][5 -_]-[5 #]'`

* **Encoding**: 

> `$ ./art -encode 'ABCDDDDDDDDDDEFG'`

* **Multi-Line & Encoding**: 

With the multi-line feature, use ./input.txt as the argument, and paste the resource in the input.txt file:

> `$ ./art -multiline -encode ./input.txt`

* **Help Message**: 

If the program is run with a -help flag: 

> `$ ./art -help`

or, if the program is run with an invalid argument number, 

the tool displays the following usage guide:

> Art Decoder/Encoder usage:<br>

> If you want to use the Web Interface, please use the server flag as follows:<br>
> `go run . -s`

> For the Command Line program, please run as follows:

> The argument should be the string to be decoded, inside single or double quotation marks, e.g.:<br>
> `go run . '[5 #][5 -_]-[5 #]'`

> For the extra features, multi-line or encoder, run the program with the corresponding flags, e.g.:<br>
> `go run . -encode '#####-_-_-_-_-_-#####'`

> When you use the multi-line feature, use ./input.txt as the argument, and paste the resource in the input.txt file:<br>
> `go run . -multiline -encode ./input.txt`

	
**Tool Functionality:**

Upon launching with valid arguments on the command line, or using the web interface, the tool functions as follows:

- A single argument inside single or double quotation marks is run from the command line, the characters are converted into the corresponding text-art, and the output is printed out in the console. The program can convert both single-line and multi-line strings of characters, as long as they are inside the same quotation marks as a single argument. 
- When the toggled-on multi-line feature is enabled, using the CLI, the user is expected to input the data to be decoded or encoded in the input.txt file. The program reads the input.txt file, converts the data, and produces the outcome in the output.txt file.
- With the toggled-on encoder mode, the program can encode decoded data from the text-art into a corresponding encoded character string.  
- Using the web interface, the user can input the source data in the text area on the page, and after pressing 'Decode' or 'Encode' buttons, the result will be printed out on the page under the Result header. 'Clear' button will refresh the page and empty both input text area and the result area.


## Examples and Screenshots:

**Single-line decoding:**

> `$ ./art '[5 #][5 -_]-[5 #]'`

Produces the following output:<br> 

> #####-_-_-_-_-_-#####<br>


**Multi-line decoding:**

Running the multi-line feature with the following data in the input.txt file:<br> 

```
[8  ]@|\[2 @]
[7  ]-[2  ][4 @]
[6  ]/7[3  ][4 @]
[5  ]/[4  ][6 @]
[5  ]\-' [8 @]`-[15 _]
[6  ]-[9 @][13  ]/[4  ]\
 [7 _]/[4  ]/_[7  ][6 _]/[6  ]|[10 _]-
/,[10 _]/  `-.[3 _]/,[13 _][10 -]_)
```

Will result in the following output in the output.txt file:<br> 

```
        @|\@@
       -  @@@@
      /7   @@@@
     /    @@@@@@
     \-' @@@@@@@@`-_______________
      -@@@@@@@@@             /    \
 _______/    /_       ______/      |__________-
/,__________/  `-.___/,_____________----------_)
```


**Web interface:**

Here is a visual of the web interface in action:

![Server Full Screenshot](./webArt/www/server_full.png)


This is the end of the README file.<br>