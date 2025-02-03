# Itinerary Prettifier

By Henna Venho
- 1st task at kood/Sisu (March 2024)


## What the Tool Does: 

**Itinerary Prettifier** is a command-line application designed for formatting flight booking data into a customer-friendly and readable format. The tool reads a text-based itinerary from an input file, which contains either flight codes, or dates or times, processes the text to make it customer-friendly, and creates an output file which contains information from converted flight codes (either the airport name or the city, depending on what is desired) and makes the date and time more readable. With an optional argument, the program will also print the formatted text to the console, with added colour, underlining and bolding. 


## Technologies Used: 

- Golang Standard Library
- Visual Studio Code
- WSL (Ubuntu)
- Version Control System (Git)
- Repository (Gitea)
- VCS Client (Bash)


## Usage: 

**The tool will be launched from the command line with three arguments:**

/1. Path to the input<br>
/2. Path to the output<br>
/3. Path to the airport lookup<br>

`-$ go run . ./input.txt ./output.txt ./airport-lookup.csv`


**Optionally, the tool can also be launched with the following flags:**

These flags are added in front of the three launch arguments, and they can all be added together, or only one at a time:

-format: to print formatted output to the console<br>
-trim: to trim the last empty line before EOF<br>
-debug: to debug white spaces by appending a specific string of vertical whitespaces into the original inputfile<br>

`-$ go run . -format -trim -debug ./input.txt ./output.txt ./airport-lookup.csv`


**-h Flag Help Message:**

If the program is run with an -h flag, as follows: 

`-$ go run . -h`

or, if the program is run with an invalid argument, or with an invalid argument number, 

the tool displays the following usage guide:

Itinerary usage:<br>
`go run . ./input.txt ./output.txt ./airport-lookup.csv`

Extra feature flags, which go before the input path:<br>
-format -trim -debug<br>

Flag usage example:<br>
`go run . -format -trim ./input.txt ./output.txt ./airport-lookup.csv`



**Tool Functionality:**
Upon launching with the valid arguments, the tool functions as follows:

- The program reads the input.txt file, where the airport data is presented as either three-letter IATA codes encoded with a single # (e.g. #LAX for Los Angeles International Airport), or as four-letter ICAO codes encoded with double ## (e.g. ##EGLL for London Heathrow Airport). 
- The program converts the aircodes flagged with # or ## to the corresponding airport names from the airport-lookup.csv file. If the codes are not found, they remain unchanged. If cities are preferred over airport names, flags in the input file should be preceded by an *, like this: *# or *##.
- If the input file, or the airport lookup file do not exist, the program displays: "Input not found", or "Airport lookup not found" respectively. 
- The program works even if the order of the columns in the airport-lookup.csv file is mixed, as long as the header row is intact within the lookup file.
- Dates and times are presented in the input file in ISO 8601 standard. The program runs them through date/time conversion, and prints them into the output file in a predetermined, readable format. If the input format is not recognized (is malformed), the lines (dates/times) remain unchanged.
- Vertical white space characters are trimmed into new-line characters, and excessive white space is reduced so that there are no more than one blank line in a row (2 new line characters) in the output. 

Optional flags: 

- If the program is run with the flag "-format", while it creates the output file and prints the information in it, at the same time it prints the output to the console terminal as well with a combination of highlighting, underlining and colour.
- If the program is run with the flag "-trim", it will trim the last empty line before EOF, if there is one.
- If the program is run with the flag "-debug", it will debug white spaces by appending a specific string of vertical whitespaces into the original inputfile.



## Examples:

**IATA and ICAO codes to airport names:**

If the input.txt file contains the following text: 

> Your flight departs from #HAJ, and your destination is ##EDDW.

The output will read: 

> Your flight departs from Hannover Airport, and your destination is Bremen Airport.


**IATA and ICAO codes to city names:**

If the input.txt file contains the following text: 

> Your flight departs from *#HAJ, and your destination is *##EDDW.

The output will read: 

> Your flight departs from Hannover, and your destination is Bremen.


**ISO dates and times to customer friendly format:**

If the input.txt file contains the following text: 

1. D(2022-05-09T08:07Z)
2. T12(2069-04-24T19:18-02:00)
3. T12(2080-05-04T14:54Z)
4. T12(1980-02-17T03:30+11:00)
5. T12(2029-09-04T03:09Z)
6. T24(2032-07-17T04:08+13:00)
7. T24(2084-04-13T17:54Z)
8. T24(2024-07-23T15:29-11:00)
9. T24(2042-09-01T21:43Z)

The output will read: 

1. 09 May 2022
2. 07:18PM (-02:00)
3. 02:54PM (+00:00)
4. 03:30AM (+11:00)
5. 03:09AM (+00:00)
6. 04:08 (+13:00)
7. 17:54 (+00:00)
8. 15:29 (-11:00)
9. 21:43 (+00:00)



This is the end of the readme file.<br>