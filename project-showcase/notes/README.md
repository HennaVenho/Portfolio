# Notes Tool 

Team:
	Miikka Valtteri Mäntysalo,
	Henna Maria Helena Venho,
	Pasi Kullervo Kuha

- Kood/Sisu Selection Sprint Group Task (February 2024)


## What the Tool Does: 

**NotesTool** is a command-line application designed for managing short single-line notes. The user can create collections of notes, view them, add new notes, or remove existing ones. Each collection operates within its own database, represented by a plain text file, ensuring that notes persist between sessions.


## Technologies Used: 

- Golang Standard Library
- Visual Studio Code
- WSL (Ubuntu)
- Version Control System (Git)
- Repository (Gitea)
- VCS Client (Bash)


## Usage: 

**To interact with a collection, use:**

-$ ./notestool <database_name>

**For example, to work with a collection named 'coding_ideas':**

-$ ./notestool coding_ideas

**This command initializes the tool, loading the 'coding_ideas' collection if it exists, or creating it if it does not.**


**Help Message:**
If you run './notestool' without arguments, or with more than one argument, the tool displays usage guide: 

-$ ./notestool
Usage: ./notestool help

which again displays a usage guide:

-$ ./notestool help
To use program please enter ./notestool database_name
if no database_name entered database_name will be notes.txt


**Tool Functionality:**
Upon launching with a valid collection name, the tool presents a menu with options to:

1. **Show Notes:** Displays all notes in the current collection.
2. **Add a Note:** Prompts the user to enter a new note, which is then added to the collection.
3. **Delete a Note:** Prompts the user for the note number to remove from the collection.
4. **Exit:** Closes the application.

After performing any operation, the user is returned to the main menu until choosing to exit.



**Data Storage:**

Each collection is stored as a plain text file named after the collection itself. Notes are stored in separate rows within this file, allowing for easy management and persistence of data.


## Examples:

**Starting the tool and interacting with a collection:**

-$ ./notestool testtag <br>
 Welcome to the Notes Tool!<br>

Please select operation: (1, 2, 3, or 4)<br>
/1. Show notes.<br>
/2. Add a note. <br>
/3. Delete a note.<br>
/4. Exit.<br>
Enter choice: 1<br>

001 - first note <br>
002 - second note<br>

Please select operation: (1, 2, 3, or 4)<br>
/1. Show notes.<br>
/2. Add a note.<br>
/3. Delete a note.<br>
/4. Exit.<br>
Enter choice: 2<br>
Enter the note text:<br>
note number three<br>

Please select operation: (1, 2, 3, or 4)<br>
/1. Show notes.<br>
/2. Add a note.<br>
/3. Delete a note.<br>
/4. Exit.<br>
Enter choice: 3<br>
Enter the number of note to remove or 0 to cancel: 2<br>

Please select operation: (1, 2, 3, or 4)<br>
/1. Show notes.<br>
/2. Add a note.<br>
/3. Delete a note.<br>
/4. Exit.<br>
Enter choice: 1<br>

001 - first note<br>
003 - note number three<br>

Please select operation: (1, 2, 3, or 4)<br>
/1. Show notes.<br>
/2. Add a note.<br>
/3. Delete a note.<br>
/4. Exit.<br>
Enter choice: 4<br>
Exiting...<br>


Adding and removing notes will update the collection file accordingly, ensuring all changes are preserved for future sessions.<br>



This is the end of the readme file.<br>