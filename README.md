# Gymnasium Database

The "Gym Management" database schema is provided below:

VILLES (CITY)

SPORTIFS (ATHLETE_ID, LAST_NAME, FIRST_NAME, GENDER, AGE, ADVISOR_ID*)

SPORTS (SPORT_ID, LABEL)

GYMS (GYM_ID, GYM_NAME, ADDRESS, CITY*, AREA)

REFEREE (ATHLETE_ID*, SPORT_ID*)

TRAINER (TRAINER_ID*, SPORT_ID*)

PLAY (ATHLETE_ID*, SPORT_ID*)

SESSIONS (GYM_ID*, SPORT_ID*, TRAINER_ID*, DAY, TIME, DURATION)
# Installation

    1. Clone the repository
    2. Install MongoDB
    3. Install MongoDB Compass
    4. Import the gym database into MongoDB Compass

# Usage

Execute the queries one by one and view the results
# License

This project is licensed under the [MIT License]. See the LICENSE.md file for more details.
# Authors

Khoukhi Aymene
