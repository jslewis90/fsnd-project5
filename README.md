[logo]: https://udacity.com/favicon.ico "Udacity"
![udacity][logo] Project 5: Design a Game
====================================


## Set-Up Instructions:
1.  Clone or download [this](https://github.com/jslewis90/fsnd-project5) repository.
2.  In app.yaml, change the app ID to one you have registed on the App Engine Admin Console.
3.  Add the project to Google App Engine Launcher by going to File -> Add Existing Application.
4.  Visit the API Explorer - by default localhost:8080/_ah/api/explorer. (Or the port specified
when you add the application.)
5.  (Optional) Deploy the application

## To Play a Game:
1.  Create a new user, using the `create_user` endpoint.
2.  Use `create_game` to create the game. Remember to copy the 'urlsafe_key' for
later use.
3.  To make a guess, use the `make_move` endpoint. Set 'guess' to your guess. If
you're guessing the word, set 'word_guess' to 'True'.
4.  If you want to end the game in its current state, use the `cancel_game`
endpoint. Note: This will count against your score.

##Score Keeping:
 - **Score Definition**
    - Scores are determined by a User's total number of wins divided by their total number of Games.
 - **Losses**
    - If a User can't guess the word within the defined 'attempts'.
    - If a User cancels a Game.
 - **Wins**
    - If a User guesses the correct word.
    - If a User guesses the word letter by letter within the defined 'attempts'.
 - **High Scores**
    - Scores are ordered by the number of guesses, which is determined by substracting 'attempts_remaining' from 'attempts_allowed'.
 - **Ranks**
    - Ranks are determined by the 'win_loss', which is stored as a float, of each User in descending order. (e.g. Highest 'win_loss' to lowest 'win_loss'.)


##Game Description:
This is a classic version of the Hangman game. A word is randomly selected from
a list of words for the player to guess. The player must guess the letters in the
word or the word itself within the specified amount of guesses to win. By default,
the number of guesses is set to 6.
Guesses are sent via the `make_move` function which will return: '{{number}} of
{{letter}}'s Found!', 'You win!', 'Game over!' (if the maximum number of attempts
is reached), 'Letter already guessed!', 'Word already guessed!', 'Only a single
letter may be guessed!' (if the user tries to guess multiple letters with word_guess
set to false).
Multiple games can be played by different users at a given time. Each game can be
interacted with by using its `urlsafe_game_key`.

##Files Included:
 - api.py: Contains endpoints and game playing logic.
 - app.yaml: App configuration.
 - cron.yaml: Cronjob configuration.
 - main.py: Handler for taskqueue handler.
 - models (`__init__.py`, game.py, message.py, score.py, user.py): Entity and message definitions including helper methods.
 - utils.py: Helper function for retrieving ndb.Models by urlsafe Key string.

##Endpoints Included:
 - **create_user**
    - Path: 'user'
    - Method: POST
    - Parameters: user_name, email (optional)
    - Returns: Message confirming creation of the User.
    - Description: Creates a new User. user_name provided must be unique. Will 
    raise a ConflictException if a User with that user_name already exists.
    
 - **new_game**
    - Path: 'game'
    - Method: POST
    - Parameters: user_name, min, max, attempts
    - Returns: GameForm with initial game state.
    - Description: Creates a new Game. user_name provided must correspond to an
    existing user - will raise a NotFoundException if not. Min must be less than
    max. Also adds a task to a task queue to update the average moves remaining
    for active games.
     
 - **get_game**
    - Path: 'game/{urlsafe_game_key}'
    - Method: GET
    - Parameters: urlsafe_game_key
    - Returns: GameForm with current game state.
    - Description: Returns the current state of a game.
    
 - **make_move**
    - Path: 'game/{urlsafe_game_key}'
    - Method: PUT
    - Parameters: urlsafe_game_key, guess, word_guess (optional)
    - Returns: GameForm with new game state.
    - Description: Accepts a 'guess' and returns the updated state of the game.
    The optional 'word_guess' will check if your guess is the word instead of
    looking for individual letters in the word. If this causes a game to end,
    a corresponding Score entity will be created.
    
 - **get_scores**
    - Path: 'scores'
    - Method: GET
    - Parameters: None
    - Returns: ScoreForms.
    - Description: Returns all Scores in the database (unordered).
    
 - **get_user_scores**
    - Path: 'scores/user/{user_name}'
    - Method: GET
    - Parameters: user_name
    - Returns: ScoreForms. 
    - Description: Returns all Scores recorded by the provided player (unordered).
    Will raise a NotFoundException if the User does not exist.
    
 - **get_active_game_count**
    - Path: 'games/active'
    - Method: GET
    - Parameters: None
    - Returns: StringMessage
    - Description: Gets the average number of attempts remaining for all games
    from a previously cached memcache key.

 - **get_user_games**
    - Path: 'games/user/{user_name}'
    - Method: GET
    - Parameters: user_name
    - Returns: GameForms
    - Description: Accepts a 'user_name' and returns the the User's active games.

 - **cancel_game**
    - Path: 'games/cancel/{urlsafe_game_key}'
    - Method: PUT
    - Parameters: urlsafe_game_key
    - Returns: GameForm
    - Description: Accepts a 'urlsafe_game_key' and cancels the Game.

 - **get_high_scores**
    - Path: 'scores/high_scores'
    - Method: GET
    - Parameters: number_of_results (optional)
    - Returns: ScoreForms
    - Description: Accepts a 'number_of_results' (optional) that limits the number of Scores returned. Otherwise, returns the top 5 Scores.

 - **get_user_rankings**
    - Path: 'scores/rankings'
    - Method: GET
    - Parameters: None
    - Returns: RankForms
    - Description: Returns all Users ordered by their win / loss ratio.

 - **get_game_history**
    - Path: 'games/history/{urlsafe_game_key}'
    - Method: GET
    - Parameters: urlsafe_game_key
    - Returns: HistoryForms
    - Description: Accepts a 'urlsafe_game_key' and returns the corresponding Game's history of moves.

##Models Included:
 - **User**
    - Stores unique user_name, (optional) email address, wins, losses, and
    win/loss (float).
    
 - **Game**
    - Stores unique game states. Associated with User model via KeyProperty.
    
 - **Score**
    - Records completed games. Associated with Users model via KeyProperty.
    
##Forms Included:
 - **GameForm**
    - Representation of a Game's state (urlsafe_key, current_word,
    attempts_remaining, game_over flag, message, user_name).
 - **NewGameForm**
    - Used to create a new game (user_name, min, max, attempts)
 - **MakeMoveForm**
    - Inbound make move form (guess).
 - **ScoreForm**
    - Representation of a completed game's Score (user_name, date, won flag,
    guesses).
 - **ScoreForms**
    - Multiple ScoreForm container.
- **RankForm**
    - Representation of a User's rank (user_name, games_won, games_lost,
    win_loss).
 - **RanksForms**
    - Multiple RankForm container.
 - **StringMessage**
    - General purpose String container.
