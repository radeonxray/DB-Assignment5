# DB-Assignment5
Database assignment regarding StoredProcedures and JSON

Assignment: https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/assignments/assignment5.md

Slides: https://github.com/datsoftlyngby/soft2019spring-databases/blob/master/lecture_notes/05-StoredProceduresAndJSON.ipynb

------
## Assignment Description

#### Exercise 1

Write a stored procedure denormalizeComments(postID) that moves all comments for a post (the parameter) into a json array on the post.

#### Exercise 2

Create a trigger such that new adding new comments to a post triggers an insertion of that comment in the json array from exercise 1.

#### Exercise 3

Rather than using a trigger, create a stored procedure to add a comment to a post - adding it both to the comment table and the json array

#### Exercise 4

Make a materialized view that has json objects with questions and its answeres, but no comments. Both the question and each of the answers must have the display name of the user, the text body, and the score.

#### Exercise 5

Using the materialized view from exercise 4, create a stored procedure with one parameter keyword, which returns all posts where the keyword appears at least once, and where at least two comments mention the keyword as well.

------

### Additional Information

Our belowed site "stackoverflow" is part of a family of technical sites called "stackexchange.com". They run over 300 discussion fora similar to stackoverflow. The data of all those sites are awailable for download at the internet archive at: https://archive.org/details/stackexchange.

The programming stackexchange is about 40GB compressed XML - I guestimate around 100GB expanded.

We will use data from stackexchange for this assignment.

I suggest you use the two dataserts for "coffee.stackexchange.com" (small - for debugging purposes), and "askubuntu.com" which is around 700MB compressed - expands into 2GB XML. You are free to choose which stackexchange site you want - if you pick something else than "askubunto" you must mention so in the readme file of the handin.

The data is in several xml files, following a fixed naming scheme. There is a text file in the right side which explains briefly each field and table.

Loading the data

[This is a script that can serve as the basis loading the files into a database](https://gist.github.com/emanoelbarreiros/c164a60e98a7482cde22)

------ 

### Hand-in

The hand-in must be handed in on peergrade. You are allowed to hand in in groups of 3. You review individually though.

The handin is a link to github. The readme file must explain where the textual representation of each of the solutions to the five handins are - it is easiest for the reviewer if they are inlined in the readme file.

------

### Setup

Assignment done using Vagrant, Docker and Workbench

Run the following command with Docker to create the container:
`docker run --name my_db5mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=iphone2019 -d mysql`

Access the Docker container:

`docker exec -it my_db5mysql bash`

Within the Docker Container, run the following 2 commands to update the container and download 7zip:
`apt-get update`
`apt-get install wget p7zip-full -y`

With the container, create a new folder and download the test-data:
`mkdir workspace`
`wget https://archive.org/download/stackexchange/askubuntu.com.7z
7z e askubuntu.com.7z  `

Start MySQL in the container:

`mysql -u root -piphone2019 --local-infile`

Run the following comman to set the local-infile:

`set global local_infile = 1;`

To connect to the Database and see the data, make sure you have the latest version of the [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)

My Default information to connect to the Docker Container:

*IP*: `192.168.33.10`

*Port*: `3306`

*User*: `root`

*Password*: `iphone2019`



### Notes - Not part of the hand-in!

```
Run the mysql shell with parameter
--local-infile

Inside run: 
set global local_infile = 1;

And then use this command to import individual files. Also change the table names and file path to your own

load xml local infile '/full/path/to/Badges.xml' into table badges rows identified by '<row>';
```
