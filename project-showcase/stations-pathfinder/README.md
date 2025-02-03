# STATIONS PATHFINDER

- Team: Henna Venho and Mikko Venäläinen 
- 4th task at kood/Sisu (June 2024)


## Introduction

**Stations Pathfinder** is a a command line program which uses path-finding algorithm to identify the quickest way to move some number of trains from a start station to an end station.


## Technologies Used: 

- Golang Standard Library
- Visual Studio Code
- WSL (Ubuntu)
- Version Control System (Git)
- Repository (Gitea)
- VCS Client (Bash)


## Tool Functionality:

* The tool uses a Dijkstra pathfinding algorithm with which it searches for the shortest path from start station to end station considering certain edges as excluded and accounting for additional costs due to other train paths. 
* The aim is to move all trains to the end station with as few turns as possible. 
* Only one train can move on any given track section between two stations at the same time, and there can only be one train at any given station at the same time, except for the start and end stations where there can be several trains at the same time. 
* There can be a maximum of 10000 stations in the network map. The network map must include a 'stations' section and a 'connections' section, and each station needs to have valid coordinates and connections.


## Building the Program

Clone the repository to your computer:

> $ git clone https://github.com/HennaVenho/stations.git

Change directory to the cloned repository:

> $ cd stations

Build the executable program (optional):

> $ go build


## Running the Program

> $ go run . [path to file containing network map] [start station] [end station] [number of trains]

For example:

> $ go run . networkmaps/network_terrains.map jungle desert 10


**Optional Arguments:**

You can see usage help with `go run . -h`


```
Stations Pathfinder usage:
$ go run . -h

Optional arguments:
-h	Show usage help
```


## Testing the Program

A suite of tests has been created to cover the test cases described on the task review page. 

To print the test results with a single PASS statement for the whole suite of tests, or possible FAIL statements if such occur, run with: 

> $ go test

To print the test results with a PASS/FAIL result for each test separately, run with: 

> $ go test -v

Both ways to run the tests will also print all the test cases with a result and an expected result. 



## Example:

network_london.map contains: 

```
stations:
waterloo,3,1
victoria,6,7
euston,11,23
st_pancras,5,15

connections:
waterloo-victoria
waterloo-euston
st_pancras-euston
victoria-st_pancras
```

Running the program with 4 trains: 

> $ go run . networkmaps/network_london.map waterloo st_pancras 4

Produces result: 

```
T1-victoria T2-euston
T1-st_pancras T2-st_pancras T3-victoria T4-euston
T3-st_pancras T4-st_pancras
```



This is the end of the README file.