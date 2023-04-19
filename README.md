# Get a Telegram notification when the balance of your LNbits wallet changes
LNbits (https://lnbits.com/) with its plethora of extensions is a great accounting system layered on top of your bitcoin holdings, but it is still young, so it lacks some basic functionality. 

One of them being a notification of incoming/outgoing payments. 

As a merchant using LNbits notifications are critical to make sure you serve your customers well and in timely fashion.

I've tried using a few mobile wallets to help with the notifications (Zeus, BitBanana, Samurai, BlueWallet, etc.), but none of them send notifications.

This short tutorial will build a python script which will check the balance of a wallet and notifies you via Telegram chat if the balance has changed.

# What you will need?

This tutorial assumes, you already know what LNbits is and how to use it as well as are familiar with Telegram.

1. LNbits instance (https://github.com/lnbits/lnbits)
2. Telegram bot (https://core.telegram.org/bots/tutorial)
3. A computer or a server where you can run a python sript and a cron

## LNbits

Instal LNbits installed on your server or a lightning node. If you want to test, just use https://legend.lnbits.com - but do not hold many funds there.

When you have your LNbits setup, you will need the 
- Invoice/Read key API (you will find that when you click on your wallet and expand the API Info section on the right
- Your API URL (example: https://legend.lnbits.com/api/v1/wallet)

To test if it works run the command under the Get wallet details section:

```
curl <ADD YOUR API URL> -H "X-Api-Key: <YOUR KEY>" 
```

Replace the <ADD YOUR API URL> and <YOUR KEY> with the values you see in your LNbits instance.
The result should be a JSON object with name and balance

{"name":"wallet name","balance":0}

## Telegram bot

When you are creating your bot, you will be asked for a name and username of your bot and after you have entered those with @BotFather, you will receive your HTTP API token. (keep it secret)

Then you will need a chat ID where you will chat with your bot. To get that, you will need to search in your Telegram app for the username you have selected for your bot. 

After you find it, click the bot and send a first message "hello world". 

Now, to retrieve the id of the chat, run this command on your computer (replacing <YOUR TOKEN> with the API token you received from Telegram earlier):

```
curl https://api.telegram.org/bot<YOUR TOKEN>/getUpdates
```

The result will be a JSON object which wil have the message and also the chat id ("chat":{"id":<SOME NUMBER>) 

## Python script

Now that we have all the information from LNbits and Telegram, we can get going with the code.

Since we will be using Telegram, you probably need to install the python package. We will use pip, the package manager for Python. If you do not have it, install it.

First check if it is installed.

```
pip --version
```

If pip is not already installed, you can install it using the following commands:

```
sudo apt-get update
sudo apt-get install python3-pip
```

This should install both Python 3 and pip. If you already have Python 3 installed, you can install pip by downloading the get-pip.py script and running it with Python:

```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py
```
Check the pip version:

```
pip --version
```
Now we'll create a couple of files. One for keeping track of the balance. The other for logging/debugging the script.
```
touch current-balance.txt
touch script.log
```

Now, we have our system ready to go. Below is the python script for a file balance.py:

```
pico -w balance.py
```
Add the following into the file:
```
#!/usr/bin/python3

import requests
import logging 
import os
import telegram
import asyncio

# Configure logging
logging.basicConfig(filename='script.log', level=logging.INFO,
                    format='%(asctime)s - %(levelname)s - %(message)s')

# Set the API URL and key
url = "https://<YOUR LNBITS URL>/api/v1/wallet"
headers = {"X-Api-Key": "<YOUR LNBITS WALLET API KEY"}

# Initialize the Telegram bot
bot = telegram.Bot(token='<YOUR TELEGRAM BOT API TOKEN')
chat_id = "<YOUR BOT CHAT ID>"

# Define the async function to send Telegram messages
async def send_telegram_message(chat_id, message):
    await bot.send_message(chat_id=chat_id, text=message)

# Get the current balance
response = requests.get(url, headers=headers)
data = response.json()
balance = data['balance']

# Check if the balance file exists
file_path = os.path.join(os.path.dirname(os.path.abspath(__file__)), 'current-balance.txt')
if not os.path.exists(file_path):
    logging.info("Balance file does not exist. Creating with current balance.")
    with open(file_path, 'w') as f:
        f.write(str(balance))
    last_balance = balance
else:
    # Read the last balance from file
    with open(file_path, 'r') as f:
       content=f.read();
       if content.strip() == '':
            logging.warning("Balance file is empty. Setting last balance to 0.")
            last_balance = 0
       else:
            last_balance = float(content)

# Check if the balance has changed
if balance != last_balance:
    # Write the new balance to file
    with open(file_path, 'w') as f:
        f.write(str(balance))
    # Log the balance change
    logging.info(f"Balance changed from {last_balance} to {balance}")

    # Send a message using Telegram
    message = f"Your LNbits balance has changed to {balance}."
    # Send the Telegram message
    asyncio.run(send_telegram_message(chat_id, message))

    logging.info(f"Bot sent")

else:
    # Log that the balance hasn't changed
    logging.info(f"Balance hasn't changed, current balance is {balance}")

```

You can remove all the logging code as that is mainly for debugging.

To run the script use the following command:

```
python3 balance.py
```

When the script notifies you correctly of any wallet balance change, you can set it up to be run automatically as you wish. 

Run the following command to open your crontab.


```
crontab -e
```

Add the following into it.

```
*/5 * * * * /usr/bin/python3 <ABSOLUTE PATH TO YOUR PYTHON FILE>/balance.py >> <ABSOLUTE PATH TO YOUR CRONLOG FILE> 2>&1
```

Don't forget to replace the path to your python file and to the location where you want the log. 
This cron will run your script every 5 mins and will output any errors into a log file.

# End Notes

This setup is just a hack to get notified. It is by no means perfect, but it will do the job.

If you found this guide useful, why not let it be known by [sending me a few sats](https://pay.360swim.com/tipjar/4) or via LN address⚡swmtr@360swim.com . 
