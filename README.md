# Scraping Images From a Website in Python

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/)

This guide explains how to scrape images from websites using Python and Selenium, enhancing data collection for machine learning, market analysis, and content curation:

- [Benefits of Image Extraction](#benefits-of-image-extraction)
- [Python Image Extraction Tutorial](#python-image-extraction-tutorial)
- [Getting Started](#getting-started)
- [Installing Selenium](#installing-selenium)
- [Connecting to the Target Website](#connecting-to-the-target-website)
- [Analyzing the Target Website](#analyzing-the-target-website)
- [Gathering Image URLs](#gathering-image-urls)
- [Downloading the Images](#downloading-the-images)
- [Complete Code Solution](#complete-code-solution)
- [Future Enhancements](#future-enhancements)

## Benefits of Image Extraction

Web scraping isn't limited to text information. You can target various data types, including multimedia files like images. Extracting images from websites serves several valuable purposes:

- Collect training data for AI and machine learning models: Download images to improve your model's accuracy and performance.
- Analyze competitors' visual marketing strategies: Help your marketing team understand trends by examining images your competitors use to convey key messages.
- Automate collection of high-quality visuals: Gather engaging images to boost your website and social media presence, helping capture and maintain audience interest.

## Python Image Extraction Tutorial

To extract images from a webpage, you'll need to follow these key steps:

1. Establish a connection with the target website
2. Identify and select relevant image HTML elements
3. Extract image URLs from these elements
4. Download the actual image files using these URLs

For this tutorial, we'll work with Unsplash, one of the web's most popular image repositories. Here's what the search results page for "wallpaper" free images looks like:

![searching for free wallpaper images](https://media.brightdata.com/2024/04/searching-for-free-wallpaper-images.gif)

Notice how the page continuously loads new images as you scroll down. This dynamic loading requires browser automation tools for effective scraping.

We'll be working with this specific URL:

```
https://unsplash.com/s/photos/wallpaper?license=free
```

## Getting Started

Before beginning, ensure you have Python 3 installed on your computer. If not, [download the installer](https://www.python.org/downloads/), run it, and follow the setup instructions.

Set up your Python image extraction project with these commands:

```sh
mkdir image-scraper
cd image-scraper
python -m venv env
```

This creates an `image-scraper` directory with a Python [virtual environment](https://docs.python.org/3/library/venv.html) inside.

Open the project folder in your preferred Python IDE. [PyCharm Community Edition](https://www.jetbrains.com/pycharm/download/#section=windows) or [Visual Studio Code with the Python extension](https://code.visualstudio.com/docs/python/python-tutorial) are excellent choices.

Create a `scraper.py` file in your project folder and add this initial code:

```python
print('Hello, World!')
```

Currently, this file simply displays "Hello, World!" but we'll soon transform it into an image extraction tool.

Test that your script works by clicking your IDE's run button or executing:

```sh
python scraper.py
```

You should see the following output in your terminal:

```
Hello, World!
```

Perfect! Your Python project is ready. Now let's implement the logic needed to extract images from websites.

## Installing Selenium

[Selenium](https://www.selenium.dev/) is an excellent library for image extraction because it handles both static and dynamic website content. As a browser automation tool, it can render pages that require JavaScript execution. Learn more in the [Selenium web scraping guide](https://brightdata.com/blog/how-tos/using-selenium-for-web-scraping).

Compared to HTML parsers like [BeautifulSoup](https://brightdata.com/blog/how-tos/beautiful-soup-web-scraping), Selenium targets more sites and supports more use cases. It's particularly effective with image providers that use interactions to load additional images—exactly what Unsplash does.

Before installing Selenium, activate your Python virtual environment. On Windows, use:

```sh
env\Scripts\activate
```

On macOS and Linux, use:

```sh
source env/bin/activate
```

In the activated environment, install the [Selenium WebDriver package](https://pypi.org/project/selenium/) with:

```sh
pip install selenium
```

Be patient during installation as it may take some time.

## Connecting to the Target Website

Import Selenium and the necessary classes to control Chrome by adding these lines to `scraper.py`:

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver.chrome.options import Options
```

Now initialize a headless Chrome WebDriver instance:

```python
# to run Chrome in headless mode
options = Options()
options.add_argument("--headless") # comment while developing

# initialize a Chrome WerbDriver instance
# with the specified options
driver = webdriver.Chrome(
    service=ChromeService(),
    options=options
)
```

Comment out the `--headless` option during development if you want Selenium to open a visible Chrome window, allowing you to monitor the script's actions in real-time. For production, keep the `--headless` option enabled to conserve resources.

Close the browser window by adding this line at the end of your script:

```python
# close the browser and free up its resources
driver.quit() 
```

To prevent issues with responsive content, maximize the Chrome window:

```python
driver.maximize_window()
```

Now instruct Chrome to navigate to the target page using Selenium's `get()` method:

```python
url = "https://unsplash.com/s/photos/wallpaper?license=free"
driver.get(url)
```

Putting it all together:

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver.chrome.options import Options

# to run Chrome in headless mode
options = Options()
options.add_argument("--headless")

# initialize a Chrome WerbDriver instance
# with the specified options
driver = webdriver.Chrome(
    service=ChromeService(),
    options=options
)

# to avoid issues with responsive content
driver.maximize_window()

# the URL of the target page
url = "https://unsplash.com/s/photos/wallpaper?license=free"
# visit the target page in the controlled browser
driver.get(url)

# close the browser and free up its resources
driver.quit()
```

Run the image extraction script in headed mode, and you'll briefly see this page before Chrome closes:

![Page shown for a fraction](https://media.brightdata.com/2024/04/Page-shown-for-a-fraction.png)

The "Chrome is being controlled by automated test software" message confirms that Selenium is successfully controlling the Chrome window.

Great! Now let's examine the HTML code to determine how to extract the images.

## Analyzing the Target Website

Before developing our image extraction logic, we need to inspect the HTML structure of our target page. This helps us define effective element selection and determine how to extract the desired data.

Visit the target site in your browser, right-click on an image, and select "Inspect" to open the DevTools:

![Inspecting an image in devtools](https://media.brightdata.com/2024/04/Inspecting-an-image-in-devtools.png)

From this inspection, we can observe two important details:

First, the image is contained in an `<img>` HTML element. The CSS selector to target these image elements is:

```python
[data-test="photo-grid-masonry-img"]
```

Second, the image elements have both the standard `src` attribute and the `srcset` attribute. If you're unfamiliar with `srcset`, it specifies multiple source images with hints to help browsers select the appropriate one based on screen size and resolution.

The `[srcset](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)` attribute value follows this format:

```python
<image_source_1_url> <image_source_1_size>, <image_source_1_url> <image_source_2_size>, ...
```

Where:

- `<image_source_1_url>`, `<image_source_2_url>`, etc. are URLs to different-sized images
- `<image_source_1_size>`, `<image_source_2_size>`, etc. indicate each image's size, expressed as pixel widths (e.g., `200w`) or pixel ratios (e.g., `1.5x`)

This dual-attribute approach is common on modern responsive websites. Directly targeting the `src` attribute isn't ideal since `srcset` often contains higher-quality image URLs.

The HTML inspection also shows that all image URLs are absolute, so we don't need to combine them with the site's base URL.

## Gathering Image URLs

Use the `findElements()` method to select all desired image elements on the page:

```python
image_html_nodes = driver.find_elements(By.CSS_SELECTOR, "[data-test=\"photo-grid-masonry-img\"]")  
```

This requires importing:

```python
from selenium.webdriver.common.by import By
```

Next, create a list to store the URLs extracted from the image elements:

```python
image_urls = []
```

Iterate through the nodes in `image_html_nodes`, retrieve the URL from either `src` or the largest image from `srcset` (if available), and add it to `image_urls`:

```python
for image_html_node in image_html_nodes:
  try:
    # use the URL in the "src" as the default behavior
    image_url = image_html_node.get_attribute("src")

    # extract the URL of the largest image from "srcset",
    # if this attribute exists
    srcset =  image_html_node.get_attribute("srcset")
    if srcset is not None:
      # get the last element from the "srcset" value
      srcset_last_element = srcset.split(", ")[-1]
      # get the first element of the value,
      # which is the image URL
      image_url = srcset_last_element.split(" ")[0]

    # add the image URL to the list
    image_urls.append(image_url)
  except StaleElementReferenceException as e:
    continue
```

Since Unsplash is highly dynamic, some images might disappear from the page during processing. To handle this, we catch the `StaleElementReferenceException`.

Remember to add this import:

```python
from selenium.common.exceptions import StaleElementReferenceException
```

Now display the extracted image URLs:

```python
print(image_urls)
```

Your current `scraper.py` file should look like:

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.common.exceptions import StaleElementReferenceException

# to run Chrome in headless mode
options = Options()
options.add_argument("--headless")

# initialize a Chrome WerbDriver instance
# with the specified options
driver = webdriver.Chrome(
    service=ChromeService(),
    options=options
)

# to avoid issues with responsive content
driver.maximize_window()

# the URL of the target page
url = "https://unsplash.com/s/photos/wallpaper?license=free"
# visit the target page in the controlled browser
driver.get(url)

# select the node images on the page
image_html_nodes = driver.find_elements(By.CSS_SELECTOR, "[data-test=\"photo-grid-masonry-img\"]")

# where to store the scraped image url
image_urls = []

# extract the URLs from each image
for image_html_node in image_html_nodes:
  try:
    # use the URL in the "src" as the default behavior
    image_url = image_html_node.get_attribute("src")

    # extract the URL of the largest image from "srcset",
    # if this attribute exists
    srcset =  image_html_node.get_attribute("srcset")
    if srcset is not None:
      # get the last element from the "srcset" value
      srcset_last_element = srcset.split(", ")[-1]
      # get the first element of the value,
      # which is the image URL
      image_url = srcset_last_element.split(" ")[0]

    # add the image URL to the list
    image_urls.append(image_url)
  except StaleElementReferenceException as e:
    continue

# log in the terminal the scraped data
print(image_urls)

# close the browser and free up its resources
driver.quit()
```

Run the script, and you'll see output similar to this:

```
[
'https://images.unsplash.com/photo-1707343843598-39755549ac9a?w=2000&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDF8MHxzZWFyY2h8MXx8d2FsbHBhcGVyfGVufDB8fDB8fHwy', 

# omitted for brevity...

'https://images.unsplash.com/photo-1507090960745-b32f65d3113a?w=2000&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MjB8fHdhbGxwYXBlcnxlbnwwfHwwfHx8Mg%3D%3D'
]
```

This array contains the URLs of the images we need to download.

## Downloading the Images

The simplest way to download images in Python is using the [`urlretrieve()`](https://docs.python.org/3/library/urllib.request.html#urllib.request.urlretrieve) method from the `urllib.request` package in the Standard Library. This function copies a network object specified by a URL to a local file.

Import `urllib.request` by adding this line at the top of your `scraper.py` file:

```python
import urllib.request
```

Create an `images` directory in your project folder:

```sh
mkdir images
```

This is where your script will save the downloaded images.

Now, loop through the list of image URLs. For each image, generate a sequential filename and download it using `urlretrieve()`:

```python
image_name_counter = 1

# download each image and add it
# to the "/images" local folder
for image_url in image_urls:
  print(f"downloading image no. {image_name_counter} ...")

  file_name = f"./images/{image_name_counter}.jpg"
  # download the image
  urllib.request.urlretrieve(image_url, file_name)

  print(f"images downloaded successfully to \"{file_name}\"\n")

  # increment the image counter
  image_name_counter += 1
```

That's all you need to download images in Python! The `print()` statements aren't required but help monitor the script's progress.

Now let's review the complete code.

## Complete Code Solution

Here's the final version of `scraper.py`:

```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.common.exceptions import StaleElementReferenceException
import urllib.request

# to run Chrome in headless mode
options = Options()
options.add_argument("--headless")

# initialize a Chrome WerbDriver instance
# with the specified options
driver = webdriver.Chrome(
    service=ChromeService(),
    options=options
)

# to avoid issues with responsive content
driver.maximize_window()

# the URL of the target page
url = "https://unsplash.com/s/photos/wallpaper?license=free"
# visit the target page in the controlled browser
driver.get(url)

# select the node images on the page
image_html_nodes = driver.find_elements(By.CSS_SELECTOR, "[data-test=\"photo-grid-masonry-img\"]")

# where to store the scraped image url
image_urls = []

# extract the URLs from each image
for image_html_node in image_html_nodes:
  try:
    # use the URL in the "src" as the default behavior
    image_url = image_html_node.get_attribute("src")

    # extract the URL of the largest image from "srcset",
    # if this attribute exists
    srcset =  image_html_node.get_attribute("srcset")
    if srcset is not None:
      # get the last element from the "srcset" value
      srcset_last_element = srcset.split(", ")[-1]
      # get the first element of the value,
      # which is the image URL
      image_url = srcset_last_element.split(" ")[0]

    # add the image URL to the list
    image_urls.append(image_url)
  except StaleElementReferenceException as e:
    continue

# to keep track of the images saved to disk
image_name_counter = 1

# download each image and add it
# to the "/images" local folder
for image_url in image_urls:
  print(f"downloading image no. {image_name_counter} ...")

  file_name = f"./images/{image_name_counter}.jpg"
  # download the image
  urllib.request.urlretrieve(image_url, file_name)

  print(f"images downloaded successfully to \"{file_name}\"\n")

  # increment the image counter
  image_name_counter += 1

# close the browser and free up its resources
driver.quit()
```

Run it with this command:

```sh
python scraper.py
```

The script will display progress messages like:

```
downloading image no. 1 ...
images downloaded successfully to "./images/1.jpg"

# omitted for brevity...

downloading image no. 20 ...
images downloaded successfully to "./images/20.jpg"
```

Check your `/images` folder to see the automatically downloaded images:

![Exploring the images folder](https://media.brightdata.com/2024/04/Exploring-the-images-folder-1.png)

Note that these images may differ from those shown in earlier screenshots because Unsplash continuously updates its content.

## Future Enhancements

While we've achieved our goal, here are some potential improvements for your Python script:

- Save image URLs to CSV or database: This allows you to reference or download them later.
- Implement download skipping: Avoid re-downloading images already in the `/images` folder to conserve bandwidth.
- Extract image metadata: Collect tags and author information for a more complete dataset. Learn how in this [Python web scraping guide](https://brightdata.com/blog/how-tos/web-scraping-with-python).
- Expand your image collection: Simulate infinite scrolling to load and download additional images.

## Summary

Extracting images from websites using Python is straightforward and requires minimal code. However, don't underestimate anti-bot systems. While Selenium is powerful, it can't overcome advanced protection technologies that might identify your script as automated and block access.

To avoid detection, you need a tool that can handle JavaScript rendering while managing fingerprinting, CAPTCHAs, and anti-scraping measures. This is exactly what [Bright Data's Scraping Browser](https://brightdata.com/products/scraping-browser) offers.

Register and start your free trial today!