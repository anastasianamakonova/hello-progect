Board Game Bot - Technical Documentation
=========================================

Project Overview
----------------

A Telegram bot that recommends board games based on user preferences.
The bot scrapes data from hobbygames.ru, stores game information in JSON
format, and provides an interactive filtering interface via Telegram
inline keyboards.

The bot supports both local Python execution and Google Colab environment.

File Structure
--------------

.. code:: text

   project_root/
   ├── board_games_bot_py.py      # Main bot application
   ├── scraper_chatbot.py          # Web scraper for game data
   ├── config_chatbot.json         # Scraper configuration
   ├── requirements.txt            # Python dependencies
   └── board_games_data/           # Generated data directory
       ├── games_for_bot.json      # Game database for bot
       ├── all_games.json          # Complete scraped data
       └── game_*.json             # Individual game files

Dependencies
------------

.. code:: text

   requests==2.31.0              # HTTP requests for scraping
   beautifulsoup4==4.12.2        # HTML parsing
   lxml==4.9.3                   # XML/HTML parser backend
   python-telegram-bot==20.7     # Telegram API wrapper
   nltk==3.8.1                   # Natural language processing
   nest_asyncio                  # Required for Google Colab

Configuration
-------------

Bot Token
^^^^^^^^^

Replace ``TOKEN`` in ``board_games_bot_py.py``:

.. code:: python

   TOKEN = "YOUR_BOT_TOKEN_HERE"

**Security Warning**: Do not commit your token to GitHub. Use environment
variables or a ``.env`` file instead.

Scraper Configuration (``config_chatbot.json``)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: json

   {
       "seed_urls": ["https://hobbygames.ru/nastolnie", ...],
       "total_articles_to_find_and_parse": 100,
       "timeout": 30,
       "request_delay": {"min": 1.0, "max": 2.5},
       "max_retries": 3
   }

+-------------------------------------+-------------------------------------+----------+
| Config parameter                    | Description                         | Type     |
+=====================================+=====================================+==========+
| ``seed_urls``                       | Entry points for crawling           | ``list`` |
+-------------------------------------+-------------------------------------+----------+
| ``total_articles_to_find_and_parse``| Target number of games to collect   | ``int``  |
+-------------------------------------+-------------------------------------+----------+
| ``timeout``                         | HTTP request timeout in seconds     | ``int``  |
+-------------------------------------+-------------------------------------+----------+
| ``request_delay``                   | Delay between requests (min/max)    | ``dict`` |
+-------------------------------------+-------------------------------------+----------+
| ``max_retries``                     | Number of retry attempts            | ``int``  |
+-------------------------------------+-------------------------------------+----------+

Classes and Methods
-------------------

BoardGameBot Class
^^^^^^^^^^^^^^^^^^

Primary class for game data management and filtering.

.. code:: python

   class BoardGameBot:
       def __init__(self)
       def load_games(self) -> List[Dict]
       def parse_players(self, players_str: str) -> Optional[tuple]
       def parse_duration(self, duration_str: str) -> Optional[int]
       def parse_age(self, age_str: str) -> Optional[int]
       def check_age_match(self, game_age: Optional[int], 
                          user_age_category: int) -> bool
       def filter_games(self, players_req, duration_req, age_req) -> List[Dict]
       def format_game_message(self, game: Dict, index: int, total: int) -> str

**Key Methods Description**

- ``parse_players()``: Extracts player range from strings like "2-4" or "2"
  Returns tuple (min, max) or None

- ``parse_duration()``: Extracts numeric duration from string
- ``parse_age()``: Extracts age restriction (e.g., "12+" → 12)

- ``check_age_match()``: Validates age compatibility
  * Category 6: 6-11 years
  * Category 12: 12-17 years
  * Category 18: 18+ years

- ``filter_games()``: Main filtering logic combining all criteria
- ``format_game_message()``: Formats game data with emojis and Markdown

Conversation States
^^^^^^^^^^^^^^^^^^^

.. code:: python

   PLAYERS = 0    # Awaiting player count
   DURATION = 1   # Awaiting duration selection
   AGE = 2        # Awaiting age selection

Conversation Handlers
^^^^^^^^^^^^^^^^^^^^^

+------------------------+-----------+-------------------------------------+
| Handler                | State     | Purpose                             |
+========================+===========+=====================================+
| ``start()``            | Entry     | Initialize conversation             |
+------------------------+-----------+-------------------------------------+
| ``ask_players()``      | -         | Display player selection buttons    |
+------------------------+-----------+-------------------------------------+
| ``players_handler()``  | PLAYERS   | Capture player count                |
+------------------------+-----------+-------------------------------------+
| ``duration_handler()`` | DURATION  | Capture time limit                  |
+------------------------+-----------+-------------------------------------+
| ``age_handler()``      | AGE       | Capture age restriction and filter  |
+------------------------+-----------+-------------------------------------+
| ``show_game()``        | -         | Display current game card           |
+------------------------+-----------+-------------------------------------+
| ``navigation_handler()``| -        | Handle prev/next/new search         |
+------------------------+-----------+-------------------------------------+
| ``cancel()``           | Fallback  | Abort conversation                  |
+------------------------+-----------+-------------------------------------+

Data Structure
--------------

Game Object Format
^^^^^^^^^^^^^^^^^^

.. code:: python

   {
       "url": "https://hobbygames.ru/game-name",
       "title": "Game Title",
       "players": "2-4",           # or "1", "2-6", etc.
       "age": "12+",               # or "6+", "18+"
       "duration": "60",           # minutes
       "type": "Strategy",         # game type
       "price": "2500",            # rubles
       "description": "Game description text...",
       "genre": "Strategy",        # categorized genre
       "parsed_at": "2026-06-12 10:30:00"
   }

Running the Application
-----------------------

Local Environment
^^^^^^^^^^^^^^^^^

.. code:: bash

   python board_games_bot_py.py

Google Colab
^^^^^^^^^^^^

The bot includes Colab-specific code:

.. code:: python

   if 'google.colab' in sys.modules:
       !pip install nest_asyncio
       !pip install python-telegram-bot --upgrade
       # ... bot runs with nest_asyncio.apply()

When running in Colab, the bot will continue running until the cell
is interrupted.

Web Scraper (``scraper_chatbot.py``)
------------------------------------

Running the Scraper
^^^^^^^^^^^^^^^^^^^

.. code:: bash

   python scraper_chatbot.py

Scraper Workflow
^^^^^^^^^^^^^^^^

1. **Discovery Phase**: Crawls catalog pages to collect game URLs
2. **Parsing Phase**: Extracts game data from each URL
3. **Storage Phase**: Saves data to JSON files

Key Extraction Functions
^^^^^^^^^^^^^^^^^^^^^^^^

+-------------------------------------+-----------------------------------------+
| Function                            | Purpose                                 |
+=====================================+=========================================+
| ``extract_game_links_from_page()``  | Find game URLs on catalog pages         |
+-------------------------------------+-----------------------------------------+
| ``extract_price()``                 | Parse price from HTML structures        |
+-------------------------------------+-----------------------------------------+
| ``extract_description()``           | Extract and clean game description      |
+-------------------------------------+-----------------------------------------+
| ``extract_game_tags()``             | Get players/duration/age from tags      |
+-------------------------------------+-----------------------------------------+
| ``get_genre_by_description()``      | Classify genre using keyword matching   |
+-------------------------------------+-----------------------------------------+

**Note**: The scraper requires at least 100 games to be collected
(configured in ``total_articles_to_find_and_parse``).

Error Handling
--------------

HTTP Request Retries
^^^^^^^^^^^^^^^^^^^^

.. code:: python

   def make_request(url: str, config: Config) -> Optional[requests.Response]:
       # Retries up to max_retries with 2-second delays between attempts

Conversation Recovery
^^^^^^^^^^^^^^^^^^^^^

- ``allow_reentry=True`` allows restarting conversation
- ``/start`` resets all user data
- Fallback handlers catch unexpected states
- ``context.user_data.clear()`` on cancel

Deployment Notes
----------------

Requirements for Production
^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. **Bot Token**: Obtain from `@BotFather <https://t.me/BotFather>`__
2. **Environment Variables**: Store token securely
3. **Data Persistence**: Ensure ``board_games_data/`` directory is writable

Security Considerations
^^^^^^^^^^^^^^^^^^^^^^^

- **CRITICAL**: Token is currently hardcoded. Move to environment variables:

.. code:: python

   import os
   TOKEN = os.getenv('BOT_TOKEN', 'your_default_token')

- Add ``.env`` to ``.gitignore``
- No user authentication implemented

Troubleshooting
---------------

Common Issues
^^^^^^^^^^^^^

+-------------------------------------+-----------------------------------------+
| Issue                               | Solution                                |
+=====================================+=========================================+
| ``File not found: games_for_bot.json`` | Run scraper first:                      |
|                                     | ``python scraper_chatbot.py``          |
+-------------------------------------+-----------------------------------------+
| Bot doesn't respond in Colab        | Make sure cell is not interrupted;      |
|                                     | the bot runs with ``await`` loop       |
+-------------------------------------+-----------------------------------------+
| Age parsing fails                   | Verify game data contains age in        |
|                                     | format "12+"                            |
+-------------------------------------+-----------------------------------------+
| Inline buttons not working          | Check callback pattern matching         |
|                                     | (``^players_``, ``^duration_``, etc.)  |
+-------------------------------------+-----------------------------------------+

Debug Mode
^^^^^^^^^^

Add to ``board_games_bot_py.py``:

.. code:: python

   import logging
   logging.basicConfig(level=logging.DEBUG)

Version History
---------------

+---------+---------+-----------------+
| Version | Date    | Changes         |
+=========+=========+=================+
| 1.0     | June 2026 | Initial release |
+---------+---------+-----------------+

License
-------

[Specify license here]

Contact
-------

[Development team contact information]