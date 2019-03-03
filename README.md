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

## Setup

#### The Docker Container
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

Download the Data **Heads-up: You might want to start making some coffee while the data is being downloaded**:

`wget https://archive.org/download/stackexchange/askubuntu.com.7z`

Extract the Data **Heads-up: You might want to start running a marathon while the data is being extracted**:

`7z e askubuntu.com.7z`

-----

#### Connect to the Database through WorkBench

To connect to the Database through Workbench, make sure you have the latest version of the [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)

My Default information to connect to the Docker Container:

*IP*: `192.168.33.10`

*Port*: `3306`

*User*: `root`

*Password*: `iphone2019`

-----

#### Setup Mysql And The Database 

Place your Terminal/Bash in the previously created Workspace-directory in your Container.

Start MySQL in the container:

`mysql -u root -piphone2019 --local-infile`

Run the following command to set the local-infile:

`set global local_infile = 1;`

Run the following long command, to setup the Database, its tables and data 

**NOTE: Executing this command will take a couple of minutes, so don't panic if you don't see the data immidiatly through Workbench**:

```mysql
create database stackoverflow DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

use stackoverflow;

create table badges (
  Id INT NOT NULL PRIMARY KEY,
  UserId INT,
  Name VARCHAR(50),
  Date DATETIME
);

CREATE TABLE comments (
    Id INT NOT NULL PRIMARY KEY,
    PostId INT NOT NULL,
    Score INT NOT NULL DEFAULT 0,
    Text TEXT,
    CreationDate DATETIME,
    UserId INT NOT NULL
);

CREATE TABLE post_history (
    Id INT NOT NULL PRIMARY KEY,
    PostHistoryTypeId SMALLINT NOT NULL,
    PostId INT NOT NULL,
    RevisionGUID VARCHAR(36),
    CreationDate DATETIME,
    UserId INT NOT NULL,
    Text TEXT
);
CREATE TABLE post_links (
  Id INT NOT NULL PRIMARY KEY,
  CreationDate DATETIME DEFAULT NULL,
  PostId INT NOT NULL,
  RelatedPostId INT NOT NULL,
  LinkTypeId INT DEFAULT NULL
);


CREATE TABLE posts (
    Id INT NOT NULL PRIMARY KEY,
    PostTypeId SMALLINT,
    AcceptedAnswerId INT,
    ParentId INT,
    Score INT NULL,
    ViewCount INT NULL,
    Body text NULL,
    OwnerUserId INT NOT NULL,
    LastEditorUserId INT,
    LastEditDate DATETIME,
    LastActivityDate DATETIME,
    Title varchar(256) NOT NULL,
    Tags VARCHAR(256),
    AnswerCount INT NOT NULL DEFAULT 0,
    CommentCount INT NOT NULL DEFAULT 0,
    FavoriteCount INT NOT NULL DEFAULT 0,
    CreationDate DATETIME
);

CREATE TABLE tags (
  Id INT NOT NULL PRIMARY KEY,
  TagName VARCHAR(50) CHARACTER SET latin1 DEFAULT NULL,
  Count INT DEFAULT NULL,
  ExcerptPostId INT DEFAULT NULL,
  WikiPostId INT DEFAULT NULL
);


CREATE TABLE users (
    Id INT NOT NULL PRIMARY KEY,
    Reputation INT NOT NULL,
    CreationDate DATETIME,
    DisplayName VARCHAR(50) NULL,
    LastAccessDate  DATETIME,
    Views INT DEFAULT 0,
    WebsiteUrl VARCHAR(256) NULL,
    Location VARCHAR(256) NULL,
    AboutMe TEXT NULL,
    Age INT,
    UpVotes INT,
    DownVotes INT,
    EmailHash VARCHAR(32)
);

CREATE TABLE votes (
    Id INT NOT NULL PRIMARY KEY,
    PostId INT NOT NULL,
    VoteTypeId SMALLINT,
    CreationDate DATETIME
);

SET GLOBAL local_infile = 1;

load XML local infile 'Badges.xml'
into table badges
rows identified by '<row>';

load XML local infile 'Comments.xml'
into table comments
rows identified by '<row>';

load XML local infile 'PostHistory.xml'
into table post_history
rows identified by '<row>';

load XML local infile 'PostLinks.xml'
into table post_links
rows identified BY '<row>';

load XML local infile 'Posts.xml'
into table posts
rows identified by '<row>';

load XML local infile 'Tags.xml'
into table tags
rows identified BY '<row>';

load XML local infile 'Users.xml'
into table users
rows identified by '<row>';

load XML local infile 'Votes.xml'
into table votes
rows identified by '<row>';

create index badges_idx_1 on badges(UserId);

create index comments_idx_1 on comments(PostId);
create index comments_idx_2 on comments(UserId);

create index post_history_idx_1 on post_history(PostId);
create index post_history_idx_2 on post_history(UserId);

create index posts_idx_1 on posts(AcceptedAnswerId);
create index posts_idx_2 on posts(ParentId);
create index posts_idx_3 on posts(OwnerUserId);
create index posts_idx_4 on posts(LastEditorUserId);

create index votes_idx_1 on votes(PostId);

ALTER TABLE `stackoverflow`.`posts` 
ADD COLUMN `Comments` JSON NULL AFTER `CreationDate`;
alter table comments modify Id int auto_increment;

```
-----

## Assignment Results

### Exercise 1

Write a stored procedure denormalizeComments(postID) that moves all comments for a post (the parameter) into a json array on the post.

**Through Workbench:**
```mysql

CREATE PROCEDURE `denormalizeComments` (p_postID INT)
BEGIN
update posts set Comments = (select json_arrayagg(Text) from comments where PostId = p_postID group by PostId) where Id = p_postID;
END

```

**Through MySQL Docker Container:**

```mysql
DROP procedure IF EXISTS `denormalizeComments`;

DELIMITER $$
CREATE PROCEDURE `denormalizeComments` (p_postID INT)
BEGIN
update posts set Comments = (select json_arrayagg(Text) from comments where PostId = p_postID group by PostId) where Id = p_postID;
END$$

DELIMITER ;

```

To check that Procedure works, execute `call denormalizeComments(1);` followed by `SELECT Id, Comments from posts where Id < 5;`


### Exercise 2

Create a trigger such that new adding new comments to a post triggers an insertion of that comment in the json array from exercise 1.


**The Command is the same, no matter if you execute it through Workbench (Standard Query) or through MySQL Docker Container**
```mysql

DELIMITER $$
DROP TRIGGER if exists insert_post_comments$$
CREATE TRIGGER insert_post_comments
AFTER INSERT ON comments
FOR EACH ROW
BEGIN
CALL denormalizeComments(NEW.PostId);
END $$
DELIMITER ;

```

To see that the trigger has been created, if you are inspecting the DB through Workbench, if you expand the `comments`-table, and also expand the `Triggers`-optin, you should now see the `insert_post_comments` 

To see the effect, insert some data like the following:

*Please Note: If you change the comment to anything positive regarding Liverpool, the Query will not work and your machine will self-destruct!*: 

`INSERT INTO comments(Id, PostId, Score, Text, CreationDate, UserId) VALUES (1 , 23, 5, "Glory Glory Man United!", NOW(), 17);`

Followed by:

`SELECT Id, Comments from posts where Id < 24;`

This Exercise made me notice, that the `comments`-table has not been set to auto-increment the `id`-column! 

### Exercise 3

Rather than using a trigger, create a stored procedure to add a comment to a post - adding it both to the comment table and the json array

**Through WorkBench:**

```mysql
CREATE DEFINER=`root`@`%` PROCEDURE `addNewComment`(p_id INT, p_postID INT, p_score INT, p_text TEXT, p_userid INT)
BEGIN
INSERT INTO comments(Id, PostId, Score, Text, CreationDate, UserId) value (p_id, p_postID, p_score, p_text, NOW(), p_userid);
END
```

**Through MySQL Docker Container:**
```mysql
USE `stackoverflow`;
DROP procedure IF EXISTS `addNewComment`;

DELIMITER $$
USE `stackoverflow`$$
CREATE DEFINER=`root`@`%` PROCEDURE `addNewComment`(p_id INT, p_postID INT, p_score INT, p_text TEXT, p_userid INT)
BEGIN
INSERT INTO comments(Id, PostId, Score, Text, CreationDate, UserId) value (p_id, p_postID, p_score, p_text, NOW(), p_userid);
END$$

DELIMITER ;
```

Test the new Procedure by running the following Query:

*Please Note: If you change the comment to anything positive regarding Liverpool, the Query will not work and your machine will automatically uploade all your embarrassing photos to Facebook!*:

`call addNewComment(2,24,8, "20 Times, 20 Times, Man United!", 17);`

See that the data has been added:

`select Id, Comments from posts where Id < 25;`


### Exercise 4

Make a materialized view that has json objects with questions and its answeres, but no comments. Both the question and each of the answers must have the display name of the user, the text body, and the score.

**[Did not finish this Exercise]**

### Exercise 5

Using the materialized view from exercise 4, create a stored procedure with one parameter keyword, which returns all posts where the keyword appears at least once, and where at least two comments mention the keyword as well.

**[Did not finish this Exercise]**

-------

## Notes - Not part of the hand-in!

```
Run the mysql shell with parameter
--local-infile

Inside run: 
set global local_infile = 1;

And then use this command to import individual files. Also change the table names and file path to your own

load xml local infile '/full/path/to/Badges.xml' into table badges rows identified by '<row>';
```

Remember to `stop` any previous containers that are running on port 3306

Command to see containers:

`docker ps`
`docker ls`
`docker container ls --all`
