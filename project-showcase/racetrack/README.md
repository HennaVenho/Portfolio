# RACETRACK

- Team: Henna Venho and Mikko Venäläinen 
- 6th task at kood/Sisu (November 2024)


## Introduction

**Racetrack** is a real-time info-screens system to control races and inform spectators.


## Technologies Used: 

- Node.JS
- Socket.IO for Real-Time Functionalities
- JSON File to Store Data
- HTML
- CSS
- Bash for Access Key Configuration
- Tunnelmole to Access the Program on All Networks
- Visual Studio Code
- WSL (Ubuntu)
- Version Control System (Git)
- Repository (Gitea)
- Gitea Projects for Task Management
- VCS Client (Bash)
- Several Browsers for Testing
- Android and IOS Phones for Testing 
- Android Tablet for Testing 


## Program Functionality

Employee interfaces which require an access key: 

* **Front-desk:** 
In the Receptionist interface you can create and remove race sessions, and add, remove, or edit race drivers, as well as see a list of upcoming race sessions. The Receptionist can assign drivers to specific cars, or auto-assign them to the next available car.

* **Race-control:** 
In the Safety Official interface you can manage race safety, start and end races, and control race modes. Finish flag is displayed when it is time for the drivers to return to the pit - this happens automatically when the time is up, but the Safety Official is also able to recall the drivers early by pushing the 'FINISH RACE' button. After the drivers have all safely returned, the Safety Official ends the race session by pushing the 'END SESSION' button. 

* **Lap-line tracker:** 
In the Lap-line Observer interface you can record laps for each car as they cross the lap line. The first lap is a warm-up lap; therefore the first lap-time to be recorded happens when the specific car's lap-line-button is pushed for the second time. 


Public displays: 

* **Leader-board:** 
This public display shows the real-time leader board for the current race, including driver rankings as well as current laps and fastest lap times per driver. The leader board also displays the remaining time on the timer and the flag colour for the current race mode. 

* **Next-race:** 
This public display shows the drivers and cars assigned to the upcoming race session.

* **Race-countdown:** 
This public display shows the timer for the remaining time of the current race session.

* **Race-flags:** 
This public display shows the race flag indicating the current race mode. Green indicates 'safe', yellow indicates 'hazard', red indicates 'danger' and it is also displayed in between races, and the chequered flag indicates finish mode where the drivers should return to the pit. 


## Launching the Program

Clone the repository to your computer:

> $ git clone https://github.com/HennaVenho/racetrack.git

Change directory to the cloned repository:

> $ cd racetrack

Install dependencies: 

> $ npm install


* The employee interfaces (/front-desk, /race-control and /lap-line-tracker) require an access key to be entered to be able to access the contents of the pages. The access keys are set up as environmental variables in the terminal when running the server. For example, run as follows with the key of your choice: 

> $ export receptionist_key=front

> $ export observer_key=lap

> $ export safety_key=race

OR run the following command:

> $ source config.sh

* For production mode with race duration of 10 mins, run:

> $ npm start

* For development mode with race duration of 1 min, run:

> $ npm run dev

You can access the interfaces locally on the computer where you are running the program, by using the localhost path in your browser: http://localhost:56789/


### **To Set up Access to the Program on All Networks:** 

We recommend using Tunnelmole to expose the local server to other networks, because Tunnelmole is free and does not require you to create any accounts in order to use it. Here is a link to Tunnelmole documentation: 

> https://tunnelmole.com/docs/

Follow these quick steps to install and run Tunnelmole: 

* You can first run the server on your computer, if you wish.

* Open a new bash in VS Code, or wherever you are accessing the command line.

* Install Tunnelmole: 

* > $ curl -O https://install.tunnelmole.com/n3d5g/install && sudo bash install

* Run Tunnelmole to Expose Your Server: 

* > $ tmole 56789

* Tunnelmole will provide a forwarding URL, such as https://mqss6h-ip-91-153-171-74.tunnelmole.net, which you can use to access the app on other devices, e.g. your mobile phone.


## Example:

Here is a visual of the racetrack info-screens program in action:

![Server Full Screenshot](./racetrack.png)


This is the end of the README file.