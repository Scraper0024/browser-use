# browser-use
Browser Use is a browser automation SDK that uses screenshots to capture the state of the browser and actions to simulate user interactions. This chapter will introduce how you can easily use browser-use to execute agent tasks on the Web with simple calls

## Get Scrapeless API Key
Go over the [**Dashboard’s Settings tab**](https://app.scrapeless.com/?utm_source=official&utm_medium=integration&utm_campaign=browser-use):

![Get Scrapeless API Key](https://assets.scrapeless.com/prod/posts/browser-use/6184965e5abc914f3d6318df9f310248.png)

Then copy and set the `SCRAPELESS_API_KEY` environment variables in your .env file.

The `OPENAI_API_KEY` environment variables in your .env file are required too.

```.env
OPENAI_API_KEY=your-openai-api-key
SCRAPELESS_API_KEY=your-scrapeless-api-key
```

> 💡 **Remember to replace the sample API key with your actual API key**

## Install Browser Use
With pip (Python>=3.11):
```Shell
pip install browser-use
```

For memory functionality (requires Python<3.13 due to PyTorch compatibility):

```Shell
pip install "browser-use[memory]"
```

## Set up Browser and Agent Configuration
Here’s how to configure the browser and create an automation agent:

```Python
from dotenv import load_dotenv
import os
import asyncio
from urllib.parse import urlencode
from langchain_openai import ChatOpenAI
from browser_use import Agent, Browser, BrowserConfig
from pydantic import SecretStr

task = "Go to Google, search for 'Scrapeless', click on the first post and return to the title"

async def setup_browser() -> Browser:
    scrapeless_base_url = "wss://browser.scrapeless.com/browser"
    query_params = {
        "token": os.environ.get("SCRAPELESS_API_KEY"),
        "session_ttl": 180,
        "proxy_country": "ANY"
    }
    browser_ws_endpoint = f"{scrapeless_base_url}?{urlencode(query_params)}"
    config = BrowserConfig(cdp_url=browser_ws_endpoint)
    browser = Browser(config)
    return browser

async def setup_agent(browser: Browser) -> Agent:
    llm = ChatOpenAI(
        model="gpt-4o", # Or choose the model you want to use
        api_key=SecretStr(os.environ.get("OPENAI_API_KEY")),
    )

    return Agent(
        task=task,
        llm=llm,
        browser=browser,
    )
```

## Create the Main Function
Here’s the main function that puts everything together:

```Python
async def main():
    load_dotenv()
    browser = await setup_browser()
    agent = await setup_agent(browser)
    result = await agent.run()
    print(result)
    await browser.close()
    
asyncio.run(main())
```

## Run your script
Run your script:

```Shell
python run main.py
```

You should see your Scrapeless session start in the [Scrapeless Dashboard](https://app.scrapeless.com/?utm_source=official&utm_medium=integration&utm_campaign=browser-use).

## Full Code
```Python
from dotenv import load_dotenv
import os
import asyncio
from urllib.parse import urlencode
from langchain_openai import ChatOpenAI
from browser_use import Agent, Browser, BrowserConfig
from pydantic import SecretStr

task = "Go to Google, search for 'Scrapeless', click on the first post and return to the title"

async def setup_browser() -> Browser:
    scrapeless_base_url = "wss://browser.scrapeless.com/browser"
    query_params = {
        "token": os.environ.get("SCRAPELESS_API_KEY"),
        "session_ttl": 180,
        "proxy_country": "ANY"
    }
    browser_ws_endpoint = f"{scrapeless_base_url}?{urlencode(query_params)}"
    config = BrowserConfig(cdp_url=browser_ws_endpoint)
    browser = Browser(config)
    return browser

async def setup_agent(browser: Browser) -> Agent:
    llm = ChatOpenAI(
        model="gpt-4o", # Or choose the model you want to use
        api_key=SecretStr(os.environ.get("OPENAI_API_KEY")),
    )

    return Agent(
        task=task,
        llm=llm,
        browser=browser,
    )

async def main():
    load_dotenv()
    browser = await setup_browser()
    agent = await setup_agent(browser)
    result = await agent.run()
    print(result)
    await browser.close()

asyncio.run(main())
```
> 💡 Browser Use currently only supports Python.

💡 You can copy the URL in **live session** to watch the session's progress in real-time, and you can also watch a replay of the session in **session history**.
