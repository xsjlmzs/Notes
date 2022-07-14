# Python Selenium使用

## 下载&安装

安装selenium

```shell
sudo pip3 install selenium
```

Selenium 需要驱动程序来与所选浏览器交互。例如，Firefox 需要 geckodriver，在运行以下示例之前需要安装它。确保它在您的 PATH 中。

| **Chrome**:  | https://chromedriver.chromium.org/downloads                  |
| ------------ | ------------------------------------------------------------ |
| **Edge**:    | https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/ |
| **Firefox**: | https://github.com/mozilla/geckodriver/releases              |
| **Safari**:  | https://webkit.org/blog/6900/webdriver-support-in-safari-10/ |

## 使用

使用火狐浏览器打开百度，输入搜索内容并回车

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys

browser = webdriver.Firefox()

browser.get('http://www.baidu.com')
assert '百度' in browser.title

elem = browser.find_element(By.CLASS_NAME, 's_ipt')  # Find the search box
elem.send_keys('seleniumhq' + Keys.RETURN)

browser.quit() # 关闭浏览器，close()方法关闭一个标签页
```

