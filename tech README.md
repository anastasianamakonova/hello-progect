markdown
# Board Game Bot — Technical Documentation

## Concept Topic

### Purpose

This documentation is intended for developers who need to understand, modify, or extend the Board Game Bot. The bot is a Telegram-based recommendation system that helps users find board games by filtering a local database based on player count, play duration, and age restrictions.

### Target Audience

- Developers maintaining or extending the bot
- Students studying bot development or web scraping
- Contributors who want to add new features
- DevOps engineers deploying the bot to production

### Context of Use

Developers will read this documentation when they need to:

- Set up the bot locally or in Google Colab
- Understand how the scraping pipeline works
- Modify filtering logic or add new criteria
- Deploy the bot to a production environment
- Debug issues with the Telegram conversation flow

The bot consists of two main components:

1. **Telegram Bot** (`board_games_bot_py.py`) — handles user interaction
2. **Web Scraper** (`scraper_chatbot.py`) — collects game data from hobbygames.ru

---

## Quick Start

### Prerequisites

- Python 3.8 or higher
- Telegram account (to create a bot token)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/board-game-bot.git
cd board-game-bot

# Install dependencies
pip install -r requirements.txt

# (Optional) For Google Colab — additional package
pip install nest_asyncio
Configuration
1. Get a Telegram Bot Token

Open Telegram and search for @BotFather

Send /newbot and follow instructions

Copy the token you receive

2. Set the token in board_games_bot_py.py

python
TOKEN = "YOUR_BOT_TOKEN_HERE"
⚠️ Security Warning: Never commit your token to GitHub! Use environment variables or a .env file for production.

Running the Bot
Local Python:

bash
python board_games_bot_py.py
Google Colab:

The bot includes special Colab handling — just run all cells. It will stay alive until you interrupt the cell.

Project Structure
text
board-game-bot/
│
├── board_games_bot_py.py      # Main bot application (Telegram)
├── scraper_chatbot.py          # Web scraper for game data
├── config_chatbot.json         # Scraper configuration
├── requirements.txt            # Python dependencies
│
└── board_games_data/           # Created by scraper
    ├── games_for_bot.json      # Game database (used by bot)
    ├── all_games.json          # Complete scraped data
    ├── game_1.json             # Individual game files
    ├── game_1_description.txt
    └── ...
Component 1: Telegram Bot (board_games_bot_py.py)
What It Does
The bot runs a conversation with the user, collects three preferences (players, duration, age), filters the game database, and displays matching games with navigation buttons.

Conversation Flow
text
/start → Ask Players → Ask Duration → Ask Age → Show Results → Navigation (Prev/Next/New Search)
Conversation States
The bot uses three states (defined as constants):

PLAYERS = 0 — waiting for player count

DURATION = 1 — waiting for duration selection

AGE = 2 — waiting for age restriction

Main Classes
BoardGameBot — Core filtering logic

Method	What It Does
load_games()	Reads games_for_bot.json
parse_players()	Converts "2-4" → (2, 4)
parse_duration()	Extracts number from "60 min" → 60
parse_age()	Extracts number from "12+" → 12
check_age_match()	Matches game age (6, 12, 18) with user category
filter_games()	Main filtering logic combining all criteria
format_game_message()	Creates pretty Markdown message with emojis
Handlers
Handler	Trigger	What It Does
start()	/start command	Initializes new session
ask_players()	After start	Displays player buttons
players_handler()	Button click	Saves choice → asks duration
duration_handler()	Button click	Saves choice → asks age
age_handler()	Button click	Filters games → shows results
show_game()	After filtering	Displays current game card
navigation_handler()	Prev/Next/New Search	Navigates or restarts
cancel()	/cancel command	Ends conversation
User Data Storage
The bot stores temporary data in context.user_data:

python
context.user_data['players']      # "4", "8+", or None
context.user_data['duration']     # "30", "60", "120", or None
context.user_data['age']          # "6", "12", "18", or None
context.user_data['filtered_games']  # List of matching games
context.user_data['current_index']   # Current position in list
context.user_data['last_message_id'] # For editing messages
Component 2: Web Scraper (scraper_chatbot.py)
What It Does
The scraper crawls hobbygames.ru, extracts board game information, and saves it to JSON files. It is used once to build the initial database.

Configuration (config_chatbot.json)
json
{
    "seed_urls": [
        "https://hobbygames.ru/nastolnie",
        "https://hobbygames.ru/nastolnie?page=2",
        ...
    ],
    "total_articles_to_find_and_parse": 100,
    "timeout": 30,
    "request_delay": {"min": 1.0, "max": 2.5},
    "max_retries": 3
}
Scraper Workflow
text
Step 1: Crawl
    Start from seed_urls → Extract all game links → Follow pagination
    ↓
Step 2: Parse
    For each game URL → Extract title, players, duration, age, price, description
    ↓
Step 3: Classify Genre
    Analyze description keywords → Assign genre (Strategy, Logic, Card, etc.)
    ↓
Step 4: Save
    Save individual JSON files + combined games_for_bot.json
Key Extraction Functions
Function	What It Extracts
extract_game_links_from_page()	URLs of game pages from catalog
extract_next_page_url()	URL of "next page" in pagination
extract_price()	Price from meta tags, data attributes, or text
extract_description()	Game description (stops at "Комплектация")
extract_game_tags()	Players, duration, age from product-tag blocks
extract_characteristics()	Fallback — reads from characteristics table
get_genre_by_description()	Keyword matching to determine genre
Running the Scraper
bash
python scraper_chatbot.py
Output: After completion, you will have:

board_games_data/games_for_bot.json — used by the bot

board_games_data/all_games.json — complete dataset

Individual game_N.json and game_N_description.txt files

Data Format
games_for_bot.json Structure
Each game object contains:

json
{
    "title": "Game Title",
    "players": "2-4",
    "age": "12+",
    "price": "2500",
    "type": "Strategy",
    "genre": "Стратегия",
    "duration": "60",
    "url": "https://hobbygames.ru/game-name",
    "description": "Full game description text..."
}
Bot's Filtering Logic
python
# Example: User selects 4 players, 60 minutes max, age 12+
filtered = bot.filter_games(players_req="4", duration_req="60", age_req="12")

# Returns only games where:
# - Players: game supports exactly 4 players (or range including 4)
# - Duration: game duration ≤ 60 minutes
# - Age: game age ≥ 12
Dependencies
Create requirements.txt with:

text
requests==2.31.0
beautifulsoup4==4.12.2
lxml==4.9.3
python-telegram-bot==20.7
nltk==3.8.1
For Google Colab, also install:

text
nest_asyncio
Common Tasks
Adding a New Filter Criterion
Add the question to the conversation flow (create new handler and state)

Store the new preference in context.user_data

Modify filter_games() method in BoardGameBot class

Update format_game_message() to display the new field

Updating the Game Database
bash
# Run the scraper to collect fresh data
python scraper_chatbot.py

# The bot will automatically use the new games_for_bot.json
Changing Genre Detection
Edit the genres dictionary in get_genre_by_description() function:

python
genres = {
    'Стратегия': ['стратег', 'тактик', 'экономик', ...],
    'НовыйЖанр': ['ключевое_слово1', 'ключевое_слово2', ...],
}
Troubleshooting
"File not found: games_for_bot.json"
Solution: Run the scraper first:

bash
python scraper_chatbot.py
Bot doesn't respond in Colab
Solution: Make sure the cell with await asyncio.Event().wait() is still running. Interrupting the cell stops the bot.

Age filter doesn't work correctly
Check: Game data must have age in format "12+", "6+", or "18+". The parser looks for pattern (\d+)\+.

Inline buttons not appearing
Solution: Update python-telegram-bot to version 20.x. Older versions have different callback handling.

Security Notes
Never commit the bot token to GitHub

Use environment variables in production:

python
import os
TOKEN = os.getenv('BOT_TOKEN')
Add .env to .gitignore if using dotenv

Version History
Version	Date	Changes
1.0	June 2026	Initial release: basic filtering, scraper, Colab support
License
[Specify your license here]

Contact
For questions or contributions, contact the development team.

text

---

## 📋 **Что изменилось:**

| Проблема | Решение |
|----------|---------|
| Таблицы плохо отображались | Убрал сложные таблицы,改用 простые списки и Markdown-таблицы с `|---|` |
| Нужен concept topic | Добавил раздел **Concept Topic** с Purpose, Target Audience, Context of Use |
| Простое описание | Упростил язык, убрал лишние детали |
| Устройство файлов | Добавил наглядную схему структуры проекта |
| Базовое взаимодействие | Описал conversation flow и как работают обработчики |

---

## 📁 **Что загружать в репозиторий:**
board-game-bot/
├── README.md # ← Техническая документация (этот файл)
├── USER_GUIDE.rst # ← Пользовательская документация (у тебя уже есть)
├── board_games_bot_py.py # ← Код бота
├── scraper_chatbot.py # ← Скрапер
├── config_chatbot.json # ← Конфиг
└── requirements.txt # ← Зависимости

text

Теперь всё красиво отображается на GitHub! 🚀
Сделай это пожалуйста уже готовым файлом, чтобы я могла скопировать в гитхаб
Вот готовый файл README.md для технической документации. Просто скопируй его целиком и вставь в GitHub:

markdown
# Board Game Bot — Technical Documentation

## Concept Topic

### Purpose

This documentation is intended for developers who need to understand, modify, or extend the Board Game Bot. The bot is a Telegram-based recommendation system that helps users find board games by filtering a local database based on player count, play duration, and age restrictions.

### Target Audience

- Developers maintaining or extending the bot
- Students studying bot development or web scraping
- Contributors who want to add new features
- DevOps engineers deploying the bot to production

### Context of Use

Developers will read this documentation when they need to:

- Set up the bot locally or in Google Colab
- Understand how the scraping pipeline works
- Modify filtering logic or add new criteria
- Deploy the bot to a production environment
- Debug issues with the Telegram conversation flow

The bot consists of two main components:

1. **Telegram Bot** (`board_games_bot_py.py`) — handles user interaction
2. **Web Scraper** (`scraper_chatbot.py`) — collects game data from hobbygames.ru

---

## Quick Start

### Prerequisites

- Python 3.8 or higher
- Telegram account (to create a bot token)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/board-game-bot.git
cd board-game-bot

# Install dependencies
pip install -r requirements.txt

# (Optional) For Google Colab — additional package
pip install nest_asyncio
Configuration
1. Get a Telegram Bot Token

Open Telegram and search for @BotFather

Send /newbot and follow instructions

Copy the token you receive

2. Set the token in board_games_bot_py.py

python
TOKEN = "YOUR_BOT_TOKEN_HERE"
⚠️ Security Warning: Never commit your token to GitHub! Use environment variables or a .env file for production.

Running the Bot
Local Python:

bash
python board_games_bot_py.py
Google Colab:

The bot includes special Colab handling — just run all cells. It will stay alive until you interrupt the cell.

Project Structure
text
board-game-bot/
│
├── board_games_bot_py.py      # Main bot application (Telegram)
├── scraper_chatbot.py          # Web scraper for game data
├── config_chatbot.json         # Scraper configuration
├── requirements.txt            # Python dependencies
│
└── board_games_data/           # Created by scraper
    ├── games_for_bot.json      # Game database (used by bot)
    ├── all_games.json          # Complete scraped data
    ├── game_1.json             # Individual game files
    ├── game_1_description.txt
    └── ...
Component 1: Telegram Bot (board_games_bot_py.py)
What It Does
The bot runs a conversation with the user, collects three preferences (players, duration, age), filters the game database, and displays matching games with navigation buttons.

Conversation Flow
text
/start → Ask Players → Ask Duration → Ask Age → Show Results → Navigation (Prev/Next/New Search)
Conversation States
The bot uses three states (defined as constants):

PLAYERS = 0 — waiting for player count

DURATION = 1 — waiting for duration selection

AGE = 2 — waiting for age restriction

Main Classes
BoardGameBot — Core filtering logic

Method	What It Does
load_games()	Reads games_for_bot.json
parse_players()	Converts "2-4" → (2, 4)
parse_duration()	Extracts number from "60 min" → 60
parse_age()	Extracts number from "12+" → 12
check_age_match()	Matches game age (6, 12, 18) with user category
filter_games()	Main filtering logic combining all criteria
format_game_message()	Creates pretty Markdown message with emojis
Handlers
Handler	Trigger	What It Does
start()	/start command	Initializes new session
ask_players()	After start	Displays player buttons
players_handler()	Button click	Saves choice → asks duration
duration_handler()	Button click	Saves choice → asks age
age_handler()	Button click	Filters games → shows results
show_game()	After filtering	Displays current game card
navigation_handler()	Prev/Next/New Search	Navigates or restarts
cancel()	/cancel command	Ends conversation
User Data Storage
The bot stores temporary data in context.user_data:

python
context.user_data['players']      # "4", "8+", or None
context.user_data['duration']     # "30", "60", "120", or None
context.user_data['age']          # "6", "12", "18", or None
context.user_data['filtered_games']  # List of matching games
context.user_data['current_index']   # Current position in list
context.user_data['last_message_id'] # For editing messages
Component 2: Web Scraper (scraper_chatbot.py)
What It Does
The scraper crawls hobbygames.ru, extracts board game information, and saves it to JSON files. It is used once to build the initial database.

Configuration (config_chatbot.json)
json
{
    "seed_urls": [
        "https://hobbygames.ru/nastolnie",
        "https://hobbygames.ru/nastolnie?page=2",
        "https://hobbygames.ru/nastolnie?page=3",
        "https://hobbygames.ru/nastolnie?page=4",
        "https://hobbygames.ru/nastolnie?page=5",
        "https://hobbygames.ru/nastolnie?page=6",
        "https://hobbygames.ru/nastolnie?page=7",
        "https://hobbygames.ru/nastolnie?page=8",
        "https://hobbygames.ru/nastolnie?page=9",
        "https://hobbygames.ru/nastolnie?page=10"
    ],
    "total_articles_to_find_and_parse": 100,
    "timeout": 30,
    "request_delay": {"min": 1.0, "max": 2.5},
    "max_retries": 3
}
Scraper Workflow
text
Step 1: Crawl
    Start from seed_urls → Extract all game links → Follow pagination
    ↓
Step 2: Parse
    For each game URL → Extract title, players, duration, age, price, description
    ↓
Step 3: Classify Genre
    Analyze description keywords → Assign genre (Strategy, Logic, Card, etc.)
    ↓
Step 4: Save
    Save individual JSON files + combined games_for_bot.json
Key Extraction Functions
Function	What It Extracts
extract_game_links_from_page()	URLs of game pages from catalog
extract_next_page_url()	URL of "next page" in pagination
extract_price()	Price from meta tags, data attributes, or text
extract_description()	Game description (stops at "Комплектация")
extract_game_tags()	Players, duration, age from product-tag blocks
extract_characteristics()	Fallback — reads from characteristics table
get_genre_by_description()	Keyword matching to determine genre
Running the Scraper
bash
python scraper_chatbot.py
Output: After completion, you will have:

board_games_data/games_for_bot.json — used by the bot

board_games_data/all_games.json — complete dataset

Individual game_N.json and game_N_description.txt files

Data Format
games_for_bot.json Structure
Each game object contains:

json
{
    "title": "Game Title",
    "players": "2-4",
    "age": "12+",
    "price": "2500",
    "type": "Strategy",
    "genre": "Стратегия",
    "duration": "60",
    "url": "https://hobbygames.ru/game-name",
    "description": "Full game description text..."
}
Bot's Filtering Logic
python
# Example: User selects 4 players, 60 minutes max, age 12+
filtered = bot.filter_games(players_req="4", duration_req="60", age_req="12")

# Returns only games where:
# - Players: game supports exactly 4 players (or range including 4)
# - Duration: game duration ≤ 60 minutes
# - Age: game age ≥ 12
Dependencies
Create requirements.txt with:

text
requests==2.31.0
beautifulsoup4==4.12.2
lxml==4.9.3
python-telegram-bot==20.7
nltk==3.8.1
For Google Colab, also install:

text
nest_asyncio
Common Tasks
Adding a New Filter Criterion
Add the question to the conversation flow (create new handler and state)

Store the new preference in context.user_data

Modify filter_games() method in BoardGameBot class

Update format_game_message() to display the new field

Updating the Game Database
bash
# Run the scraper to collect fresh data
python scraper_chatbot.py

# The bot will automatically use the new games_for_bot.json
Changing Genre Detection
Edit the genres dictionary in get_genre_by_description() function:

python
genres = {
    'Стратегия': ['стратег', 'тактик', 'экономик', ...],
    'НовыйЖанр': ['ключевое_слово1', 'ключевое_слово2', ...],
}
Troubleshooting
Problem	Solution
"File not found: games_for_bot.json"	Run scraper first: python scraper_chatbot.py
Bot doesn't respond in Colab	Make sure the cell with await asyncio.Event().wait() is still running
Age filter doesn't work correctly	Game data must have age in format "12+", "6+", or "18+"
Inline buttons not appearing	Update python-telegram-bot to version 20.x
Security Notes
Never commit the bot token to GitHub

Use environment variables in production:

python
import os
TOKEN = os.getenv('BOT_TOKEN')
Add .env to .gitignore if using dotenv

Version History
Version	Date	Changes
1.0	June 2026	Initial release: basic filtering, scraper, Colab support
License
[Specify your license here]

Contact
For questions or contributions, contact the development team.

text

---

Просто скопируй этот код и вставь в файл `README.md` в твоём репозитории на GitHub. Всё будет красиво отображаться! 🚀
