# freeCodeCamp: Build a world cup database

## Instructions

### Part 1: Create the database
Log into the psql interactive terminal with psql --username=freecodecamp --dbname=postgres and create your database structure according to the user stories below. Don't forget to connect to the database after you create it.

### Part 2: Insert the data
Complete the `insert_data.sh` script to correctly insert all the data from `games.csv` into the database. 

- The file is started for you. Do not modify any of the code you start with. Using the `PSQL` variable defined, you can make database queries like this: `$($PSQL "<query_here>")`.
- The tests have a 20 second limit, so try to make your script efficient. The less you have to query the database, the faster it will be. You can empty the rows in the tables of your database with `TRUNCATE TABLE games, teams;`

### Part 3: Query the database
Complete the empty `echo` commands in the `queries.sh` file to produce output that matches the `expected_output.txt` file. 

- The file has some starter code, and the first query is completed for you. Use the `PSQL` variable defined to complete rest of the queries.
- **Note** that you need to have your database filled with the correct data from the script to get the correct results from your queries.

  

## Requirements & My Solutions
- You should create a database named worldcup
```
# Connect to the database
psql --username=freecodecamp --dbname=postgres

# Create a database named worldcup
CREATE DATABASE worldcup;
```

- You should connect to your worldcup database and then create teams and games tables
```
# Connect to database
\c worldcup

# Create table 'teams' and 'games'
CREATE TABLE teams();
CREATE TABLE games();
```

- Your teams table should have a team_id column that is a type of SERIAL and is the primary key, and a name column that has to be UNIQUE
```
# Alter table 'teams' -  Add columns 'team_id' and 'name'
ALTER TABLE teams ADD COLUMN team_id SERIAL PRIMARY KEY NOT NULL;
ALTER TABLE teams ADD COLUMN name VARCHAR(255) UNIQUE NOT NULL;
```

- Your games table should have a game_id column that is a type of SERIAL and is the primary key, a year column of type INT, and a round column of type VARCHAR
```
# Alter table 'games' - Add columns 'game_id', 'year', 'round'
ALTER TABLE games ADD COLUMN game_id SERIAL PRIMARY KEY NOT NULL;
ALTER TABLE games ADD COLUMN year INT NOT NULL;
ALTER TABLE games ADD COLUMN round VARCHAR(255) NOT NULL;
```

- Your games table should have winner_id and opponent_id foreign key columns that each reference team_id from the teams table
- Your games table should have winner_goals and opponent_goals columns that are type INT
- All of your columns should have the NOT NULL constraint
```
# Alter table 'games' - Add columns 'winner_id' and 'opponent_id'
ALTER TABLE games ADD COLUMN winner_id INT NOT NULL;
ALTER TABLE games ADD COLUMN opponent_id INT NOT NULL;

# Alter table 'games' - Add foreign keys 'winnder_id' and 'opponent_id'
ALTER TABLE games ADD FOREIGN KEY(winner_id) REFERENCES teams(team_id);
ALTER TABLE games ADD FOREIGN KEY(opponent_id) REFERENCES teams(team_id);

# Alter table 'games' - Add columns 'winner_goals' and 'opponent_goals'
ALTER TABLE games ADD COLUMN winner_goals INT NOT NULL;
ALTER TABLE games ADD COLUMN opponent_goals INT NOT NULL;
```

- Your two script (.sh) files should have executable permissions. Other tests involving these two files will fail until permissions are correct. When these permissions are enabled, the tests will take significantly longer to run
```
## Here is the SQL query script file:
#! /bin/bash

if [[ $1 == "test" ]]
then
  PSQL="psql --username=postgres --dbname=worldcuptest -t --no-align -c"
  echo "Test"
else
  PSQL="psql --username=freecodecamp --dbname=worldcup -t --no-align -c"
  echo "Prod"
fi

# Read the CSV and process each game
cat games.csv | while IFS="," read YEAR ROUND WINNER OPPONENT WINNER_GOALS OPPONENT_GOALS
do
  if [[ $YEAR != "year" ]]
  then
    # Get winner_id
    WINNER_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$WINNER'")
    # If not found, insert the winner
    if [[ -z $WINNER_ID ]]
    then
      $PSQL "INSERT INTO teams(name) VALUES ('$WINNER')"
      WINNER_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$WINNER'")
    fi

    # Get opponent_id
    OPPONENT_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$OPPONENT'")
    # If not found, insert the opponent
    if [[ -z $OPPONENT_ID ]]
    then
      $PSQL "INSERT INTO teams(name) VALUES ('$OPPONENT')"
      OPPONENT_ID=$($PSQL "SELECT team_id FROM teams WHERE name='$OPPONENT'")
    fi

    # Insert game details
    $PSQL "INSERT INTO games(winner_id, opponent_id, winner_goals, opponent_goals, year, round) VALUES ($WINNER_ID, $OPPONENT_ID, $WINNER_GOALS, $OPPONENT_GOALS, $YEAR, '$ROUND')"
  fi
done
```

- When you run your `insert_data.sh` script, it should add each unique team to the `teams` table. There should be 24 rows
- When you run your `insert_data.sh` script, it should insert a row for each line in the `games.csv` file (other than the top line of the file). There should be 32 rows. Each row should have every column filled in with the appropriate info. Make sure to add the correct ID's from the teams table (you cannot hard-code the values)
```
# Run this script in a new terminal to open the script file
chmod +x insert_data.sh

# Run the script file
./insert_data.sh
```

- You should correctly complete the queries in the queries.sh file. Fill in each empty echo command to get the output of what is suggested with the command above it. Only use a single line like the first query.

```
#! /bin/bash

PSQL="psql --username=freecodecamp --dbname=worldcup --no-align --tuples-only -c"

# Do not change code above this line. Use the PSQL variable above to query your database.

echo -e "\nTotal number of goals in all games from winning teams:"
echo "$($PSQL "SELECT SUM(winner_goals) FROM games")"

echo -e "\nTotal number of goals in all games from both teams combined:"
echo "$($PSQL "SELECT SUM(winner_goals + opponent_goals) FROM games")"

echo -e "\nAverage number of goals in all games from the winning teams:"
echo "$($PSQL "SELECT AVG(winner_goals) FROM games")"

echo -e "\nAverage number of goals in all games from the winning teams rounded to two decimal places:"
echo "$($PSQL "SELECT ROUND(AVG(winner_goals), 2) FROM games")"

echo -e "\nAverage number of goals in all games from both teams:"
echo "$($PSQL "SELECT AVG(winner_goals + opponent_goals) FROM games")"

echo -e "\nMost goals scored in a single game by one team:"
echo "$($PSQL "SELECT MAX(GREATEST(winner_goals, opponent_goals)) FROM games")"

echo -e "\nNumber of games where the winning team scored more than two goals:"
echo "$($PSQL "SELECT COUNT(*) FROM games WHERE winner_goals > 2")"

echo -e "\nWinner of the 2018 tournament team name:"
echo "$($PSQL "SELECT name FROM teams JOIN games ON teams.team_id = games.winner_id WHERE year = 2018 AND round = 'Final'")"

echo -e "\nList of teams who played in the 2014 'Eighth-Final' round:"
echo "$($PSQL "SELECT DISTINCT(name) FROM teams JOIN games ON teams.team_id = games.winner_id OR teams.team_id = games.opponent_id WHERE year = 2014 AND round = 'Eighth-Final' ORDER BY name")"

echo -e "\nList of unique winning team names in the whole data set:"
echo "$($PSQL "SELECT DISTINCT(name) FROM teams JOIN games ON teams.team_id = games.winner_id ORDER BY name")"

echo -e "\nYear and team name of all the champions:"
echo "$($PSQL "SELECT year || '|' || name FROM games JOIN teams ON teams.team_id = games.winner_id WHERE year IN (2014, 2018) AND round = 'Final' ORDER BY year")"

echo -e "\nList of teams that start with 'Co':"
echo "$($PSQL "SELECT name FROM teams WHERE name ILIKE 'Co%' ORDER BY name")"
```

- The output should match what is in the expected_output.txt file exactly, take note of the number of decimal places in some of the query results
```
# Run this script in a new terminal to open the script file
chmod +x queries.sh

# Run the script file
./queries.sh
```

  
