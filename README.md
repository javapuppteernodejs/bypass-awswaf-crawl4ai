# 🚀 Bypassing AWS WAF: Crawl4AI & CapSolver Integration

## 🌟 Project Introduction

This project offers a robust solution for developers to overcome **AWS Web Application Firewall (WAF)** challenges using[Crawl4AI](https://docs.crawl4ai.com/), an advanced web crawler, with [CapSolver](https://dashboard.capsolver.com/passport/login/?utm_source=github&utm_medium=partnership&utm_campaign=how-to-solve-awswaf-in-crawl4ai-capsolver). AWS WAF is a critical security service that protects web applications from common exploits, but it often blocks legitimate automated web scraping and data extraction processes. As an individual developer, I've encountered the complexities of WAFs and developed this integration to ensure seamless and uninterrupted web automation tasks.

## ✨ Core Features

-   **CapSolver `AntiAwsWafTaskProxyLess` Integration**: Utilize CapSolver's API to obtain the necessary `aws-waf-token` cookie.
-   **Crawl4AI JavaScript Injection**: Inject the obtained `aws-waf-token` as a cookie into Crawl4AI's browser context and reload the page.
-   **Browser Extension Integration**: Support for automatic AWS WAF solving via CapSolver's browser extension within a persistent Crawl4AI browser context.
-   **Seamless AWS WAF Bypass**: Enable Crawl4AI to navigate WAF-protected sites efficiently.
-   **Python Implementation**: Clear, executable Python code examples for both API and browser extension integration methods.

## ⚙️ How It Works

### Method 1: CapSolver API Integration

1.  **Initial Navigation**: Crawl4AI attempts to access the target webpage protected by AWS WAF.
2.  **Identify WAF Challenge**: The script recognizes the need for an `aws-waf-token`.
3.  **Obtain AWS WAF Cookie**: Call CapSolver's API (`AntiAwsWafTaskProxyLess`) with `websiteURL` to get the `aws-waf-token` cookie.
4.  **Inject Cookie & Reload**: Use Crawl4AI's `js_code` to set the `aws-waf-token` as a cookie and trigger `location.reload()`.
5.  **Continue Operations**: Crawl4AI proceeds with scraping after successful bypass.

### Method 2: CapSolver Browser Extension Integration

1.  **Persistent Browser Context**: Configure Crawl4AI with `user_data_dir` to launch a browser instance that retains the installed CapSolver extension.
2.  **Install & Configure Extension**: Manually install and configure the CapSolver extension (with your API key) in this browser profile for automatic AWS WAF solving.
3.  **Navigate & Auto-Solve**: Crawl4AI navigates to the target page, and the CapSolver extension automatically detects and solves the AWS WAF challenge, applying the `aws-waf-token` cookie.
4.  **Proceed**: Crawl4AI continues its tasks in the now-verified browser context.

## 🚀 Getting Started

### 🛠️ Environment Setup

Install the necessary libraries:

```bash
pip install capsolver crawl4ai
```

### 🔑 Configure Your API Key

Replace `api_key` with your CapSolver API key from the [CapSolver Dashboard](https://dashboard.capsolver.com/).

```python
# TODO: Set your configuration
api_key = "CAP-xxxxxxxxxxxxxxxxxxxxx"  # Your CapSolver API key
```

### 💻 Example Code: API Integration

See `api_integration_aws_waf.py` for the full example.

```python
import asyncio
import capsolver
from crawl4ai import *


# TODO: Set your configuration
api_key = "CAP-xxxxxxxxxxxxxxxxxxxxx"              # Your CapSolver API key
site_url = "https://nft.porsche.com/onboarding@6"  # Page URL of your target site
cookie_domain = ".nft.porsche.com"                 # The domain name to which you want to apply the cookie
captcha_type = "AntiAwsWafTaskProxyLess"           # Type of your target captcha
capsolver.api_key = api_key


async def main():
    browser_config = BrowserConfig(
        verbose=True,
        headless=False,
        use_persistent_context=True,
    )

    async with AsyncWebCrawler(config=browser_config) as crawler:
        await crawler.arun(
            url=site_url,
            cache_mode=CacheMode.BYPASS,
            session_id="session_captcha_test"
        )

        # Get AWS WAF cookie using CapSolver SDK
        solution = capsolver.solve({
            "type": captcha_type,
            "websiteURL": site_url,
        })
        cookie = solution["cookie"]
        print("aws waf cookie:", cookie)

        js_code = """
            document.cookie = \'aws-waf-token=""" + cookie + """;domain=""" + cookie_domain + """;path=/\';
            location.reload();
        """

        wait_condition = """() => {
            return document.title === \'Join Porsche’s journey into Web3\';
        }"""

        run_config = CrawlerRunConfig(
            cache_mode=CacheMode.BYPASS,
            session_id="session_captcha_test",
            js_code=js_code,
            js_only=True,
            wait_for=f"js:{wait_condition}"
        )

        result_next = await crawler.arun(
            url=site_url,
            config=run_config,
        )
        print(result_next.markdown)


if __name__ == "__main__":
    asyncio.run(main())
```

### 💡 Code Analysis (API Integration)

1.  **CapSolver SDK Call:** The `capsolver.solve` method is invoked with `AntiAwsWafTaskProxyLess` type and `websiteURL` to retrieve the `aws-waf-token` cookie. This is the crucial step where CapSolver's AI solves the WAF challenge and provides the necessary cookie.
2.  **JavaScript Injection (`js_code`):** The `js_code` string contains JavaScript that sets the `aws-waf-token` cookie in the browser's context using `document.cookie`. It then triggers `location.reload()` to refresh the page, ensuring that the subsequent request includes the newly set valid cookie.
3.  **`wait_for` Condition:** A `wait_condition` is defined to ensure Crawl4AI waits for the page title to match 'Join Porsche’s journey into Web3', indicating that the WAF has been successfully bypassed and the intended content is loaded.

## Integration Method 2: CapSolver Browser Extension Integration

CapSolver's browser extension provides a simplified approach for handling AWS WAF challenges, especially when leveraging its automatic solving capabilities within a persistent browser context managed by Crawl4AI.

### How it Works:

1.  **Persistent Browser Context:** Configure Crawl4AI to use a `user_data_dir` to launch a browser instance that retains the installed CapSolver extension and its configurations.
2.  **Install and Configure Extension:** Manually install the CapSolver extension into this browser profile and configure your CapSolver API key. The extension can be set to automatically solve AWS WAF challenges.
3.  **Navigate to Target Page:** Crawl4AI navigates to the webpage protected by AWS WAF.
4.  **Automatic Solving:** The CapSolver extension, running within the browser context, detects the AWS WAF challenge and automatically obtains the `aws-waf-token` cookie. This cookie is then automatically applied to subsequent requests.
5.  **Proceed with Actions:** Once AWS WAF is solved by the extension, Crawl4AI can continue with its scraping tasks, as the browser context will now have the necessary valid cookies for subsequent requests.

### 💻 Example Code: Extension Integration (Automatic Solving)

See `extension_integration_aws_waf_auto.py` for the full example.

```python
import asyncio
import time

from crawl4ai import *


# TODO: Set your configuration
user_data_dir = "/browser-profile/Default1" # Ensure this path is correctly set and contains your configured extension

browser_config = BrowserConfig(
    verbose=True,
    headless=False,
    user_data_dir=user_data_dir,
    use_persistent_context=True,
    proxy="http://127.0.0.1:13120", # Optional: configure proxy if needed
)

async def main():
    async with AsyncWebCrawler(config=browser_config) as crawler:
        result_initial = await crawler.arun(
            url="https://nft.porsche.com/onboarding@6", # Use the target URL protected by AWS WAF
            cache_mode=CacheMode.BYPASS,
            session_id="session_captcha_test"
        )

        # The extension will automatically solve the AWS WAF upon page load.
        # You might need to add a wait condition or time.sleep for the WAF to be solved
        # before proceeding with further actions.
        time.sleep(30) # Example wait, adjust as necessary for the extension to operate

        # Continue with other Crawl4AI operations after AWS WAF is solved
        # For instance, check for elements or content that appear after successful verification
        # print(result_initial.markdown) # You can inspect the page content after the wait


if __name__ == "__main__":
    asyncio.run(main())
```

### 💡 Code Analysis (Automatic Extension Integration)

1.  **`user_data_dir`**: This parameter is essential for Crawl4AI to launch a browser instance that retains the installed CapSolver extension and its configurations. Ensure the path points to a valid browser profile directory where the extension is installed.
2.  **Automatic Solving**: The CapSolver extension is designed to automatically detect and solve AWS WAF challenges. A `time.sleep` is included as a general placeholder to allow the extension to complete its background operations. For more robust solutions, consider using Crawl4AI's `wait_for` functionality to check for specific page changes that indicate successful AWS WAF resolution.

### 💻 Example Code: Extension Integration (Manual Solving)

See `extension_integration_aws_waf_manual.py` for the full example.

```python
import asyncio
import time

from crawl4ai import *


# TODO: Set your configuration
user_data_dir = "/browser-profile/Default1" # Ensure this path is correctly set and contains your configured extension

browser_config = BrowserConfig(
    verbose=True,
    headless=False,
    user_data_dir=user_data_dir,
    use_persistent_context=True,
    proxy="http://127.0.0.1:13120", # Optional: configure proxy if needed
)

async def main():
    async with AsyncWebCrawler(config=browser_config) as crawler:
        result_initial = await crawler.arun(
            url="https://nft.porsche.com/onboarding@6", # Use the target URL protected by AWS WAF
            cache_mode=CacheMode.BYPASS,
            session_id="session_captcha_test"
        )

        # Wait for a moment for the page to load and the extension to be ready
        time.sleep(6)

        # Use js_code to trigger the manual solve button provided by the CapSolver extension
        js_code = """
            let solverButton = document.querySelector(\'#capsolver-solver-tip-button\');
            if (solverButton) {
            // click event
                const clickEvent = new MouseEvent(\'click\', {
                    bubbles: true,
                    cancelable: true,
                    view: window
                });
                
                solverButton.dispatchEvent(clickEvent);
            }
        """
        print(js_code)
        run_config = CrawlerRunConfig(
            cache_mode=CacheMode.BYPASS,
            session_id="session_captcha_test",
            js_code=js_code,
            js_only=True,
        )
        result_next = await crawler.arun(
            url="https://nft.porsche.com/onboarding@6",
            config=run_config
        )
        print("JS Execution results:", result_next.js_execution_result)

        # Allow time for the AWS WAF to be solved after manual trigger
        time.sleep(30) # Example wait, adjust as necessary

        # Continue with other Crawl4AI operations


if __name__ == "__main__":
    asyncio.run(main())
```

### 💡 Code Analysis (Manual Extension Integration)

1.  **`manualSolving`**: Before running this code, ensure the CapSolver extension's `config.js` has `manualSolving` set to `true`.
2.  **Triggering Solve**: The `js_code` simulates a click event on the `#capsolver-solver-tip-button`, which is the button provided by the CapSolver extension for manual solving. This gives you precise control over when the AWS WAF resolution process is initiated.

## 🤝 Contributing

Contributions are welcome! Feel free to submit Issues or Pull Requests if you have improvements or find bugs.

## 📄 License

This project is licensed under the MIT License. See the `LICENSE` file for details.

## 🔗 References

-   [Crawl4AI Official Documentation](https://docs.crawl4ai.com/)
-   [CapSolver Official Documentation](https://docs.capsolver.com/)
-   [CapSolver: AWS WAF documentation](https://docs.capsolver.com/guide/captcha/awsWaf/)
-   [Overall Crawl4AI CapSolver Integration Blog Post](https://www.capsolver.com/blog/Partners/crawl4ai-capsolver)

