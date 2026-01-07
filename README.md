# Regex를 사용한 Webスクレイピング

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 [Python for web scraping](https://brightdata.co.kr/blog/how-tos/web-scraping-with-python)에서 정규 표현식을 사용하는 방법을 설명합니다:

- [정규 표현식 이해하기](#understanding-regular-expressions)
- [Webスクレイピング을 위한 Python에서의 Regex 구현](#implementing-regex-in-python-for-web-scraping)
- [Webスクレイピング에서 Regex 사용의 제약 사항](#constraints-of-using-regex-for-web-scraping)

## What Is Regex

정규 표현식(regex)은 텍스트에서 정보를 추출하기 위한 강력한 패턴 매칭 공식으로, Webスクレイピング에 유용한 도구입니다. 이러한 표현식은 텍스트 내에서 일치시킬 특정 패턴을 정의하여, 정밀한 정보 추출을 가능하게 합니다.

Python에서 정규 표현식은 특정 패턴을 매칭하기 위해 토큰을 사용합니다. 모든 토큰을 다루는 것은 이 글의 범위를 벗어나지만, 아래는 자주 사용되며 접하게 될 몇 가지입니다:

| Token | Matches |
| --- | --- |
| Any non-special character | The character given |
| `^` | Start of a string |
| `$` | End of a string |
| `.` | Any character except `\n` |
| `*` | Zero or more occurrences of the previous element |
| `?` | Zero or one occurrence of the previous element |
| `+` | One or more occurrences of the previous characters |
| `{Digit}` | Exact number of the previous element |
| `\d` | Any digit |
| `\s` | Any whitespace character |
| `\w` | Any word character |
| `\D` | Inverse of `\d` |
| `\S` | Inverse of `\s` |
| `\W` | Inverse of `\w` |

직접 실습하며 regex를 더 알아보려면 [regexr.com](https://regexr.com/)을 방문하십시오. 또한 [이 글](https://docs.bmc.com/docs/discovery/113/improving-the-performance-of-regular-expressions-788111995.html)에는 regex 성능을 최적화하기 위한 핵심 팁이 제공됩니다.

## Using Regex in Python for Web Scraping

이 섹션에서는 regex를 사용하여 다양한 웹사이트에서 데이터를 추출하는 기본 Python Webスクレイ퍼를 개발하겠습니다.

먼저 프로젝트 디렉터리를 생성합니다:

```sh
mkdir web_scraping_with_regex
cd web_scraping_with_regex
```

그다음 Python 가상 환경을 생성합니다:

```sh
python -m venv venv
```

활성화합니다:

```sh
source ./venv/bin/activate
```

Windows에서는 다음을 실행합니다:

```
venv\Scripts\activate
```

이 Webスクレイ퍼에는 두 개의 라이브러리가 필요합니다:

- 웹 페이지를 가져오기 위한 `requests`
- HTML 콘텐츠를 파싱하고 요소를 locting하기 위한 `beautifulsoup4`

라이브러리를 설치합니다:

```sh
pip install beautifulsoup4 requests
```

> **Note:**
> スクレイピング하기 전에 항상 웹사이트의 약관을 확인하여 허용되는지 확인하십시오. 명시적으로 금지되어 있다면 スクレイピング을 피하십시오.

### Scraping an E-commerce Site

[더미 이커머스 사이트](https://books.toscrape.com/)에서 책 제목과 가격을 추출하는 スクレイ퍼를 만들어 보겠습니다. 첫 페이지를 スクレイ핑하여 책의 제목과 가격을 추출합니다.

`scraper.py`라는 파일을 만들고 필요한 모듈을 import합니다:

```python
import requests
from bs4 import BeautifulSoup
import re
```

> **Note:** `re` 모듈은 regex를 다루는 Python 내장 모듈입니다.

다음으로 GET リクエスト를 보내 웹 페이지의 HTML 콘텐츠를 가져옵니다:


```python
page = requests.get('https://books.toscrape.com/')
```

이 데이터를 Beautiful Soup에 전달하여 HTML 구조를 파싱합니다:

```python
soup = BeautifulSoup(page.content, 'html.parser')
```

요소가 HTML에서 어떻게 구성되어 있는지 파악하기 위해 **Inspect Element** 도구를 사용합니다. 브라우저에서 [웹 페이지](https://books.toscrape.com/)를 열고 **Ctrl + Shift + I**를 눌러 **Inspector**를 여십시오. 스크린샷에서 볼 수 있듯이, 상품은 class가 `col-xs-6 col-sm-4 col-md-3 col-lg-3`인 `li` 요소에 저장됩니다. 책 제목은 `a` 요소의 `title` 속성을 읽어 얻을 수 있으며, 가격은 class가 `price_color`인 `p` 요소에 저장됩니다:

![prices are stored in p elements with class price_color](https://github.com/luminati-io/web-scraping-with-regex/blob/main/images/prices-are-stored-in-p-elements-with-class-price_color.png)

Beautiful Soup의 `find_all` 메서드를 사용하여 class가 `col-xs-6 col-sm-4 col-md-3 col-lg-3`인 모든 `li` 요소를 찾습니다:

```python
books = soup.find_all("li", class_="col-xs-6 col-sm-4 col-md-3 col-lg-3")
content = str(books)
```

이제 `content` 변수에는 `li` 요소의 HTML 텍스트가 들어 있으며, regex를 사용해 제목과 가격을 추출할 수 있습니다.

먼저 HTML 구조를 다시 살펴 책 제목과 가격에 매칭되는 regex 패턴을 생성합니다.

책 제목은 `a` 요소의 `title` 속성에 있으며 다음과 같은 형태입니다:

```html
<a href="..." title="...">
```

title 뒤의 큰따옴표 안 콘텐츠를 캡처하려면 `.*?` regex 패턴을 사용합니다. 여기서 `.`는 어떤 문자든 매칭하고, `*`는 이전 요소의 0회 이상 반복을 매칭하며, `?`는 매칭을 비탐욕(non-greedy)으로 만듭니다. 전체 표현식은 다음과 같습니다:

```regex
<a href=".*?" title="(.*?)"
```

`.*?` 주위의 괄호는 [capturing group](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Regular_expressions/Capturing_group)을 생성하며, 매칭된 콘텐츠를 나중에 사용할 수 있도록 저장합니다.

가격도 유사한 접근을 사용합니다. 가격은 class가 `price_color`인 `p` 요소에 나타나므로 regex 패턴은 `<p class="price_color">(.*?)</p>`입니다.

두 패턴을 정의합니다:

```python
re_book_title = r'<a href=".*?" title="(.*?)"'
re_prices = r'<p class="price_color">(.*?)</p>'
```

> **Note:**
> `.*` 뒤에 `?`가 왜 필요한지 궁금하다면, 이 [Stack Overflow 답변](https://stackoverflow.com/a/2301298)이 `?`의 역할을 잘 설명합니다.

이제 `re.findall()`을 사용해 HTML 문자열에서 모든 regex 매치를 찾습니다:

```python
titles = re.findall(re_book_title, content)
prices = re.findall(re_prices, content)
```

마지막으로 매치를 순회하며 결과를 출력합니다:

```python
for i in zip(titles, prices):
    print(f"{i[0]}: {i[1]}")
```

`python scraper.py`로 이 코드를 실행하십시오. 예상 출력은 다음과 같습니다:

```
A Light in the Attic: £51.77
Tipping the Velvet: £53.74
Soumission: £50.10
Sharp Objects: £47.82
Sapiens: A Brief History of Humankind: £54.23
The Requiem Red: £22.65
The Dirty Little Secrets of Getting Your Dream Job: £33.34
The Coming Woman: A Novel Based on the Life of the Infamous Feminist, Victoria Woodhull: £17.93
The Boys in the Boat: Nine Americans and Their Epic Quest for Gold at the 1936 Berlin Olympics: £22.60
The Black Maria: £52.15
Starving Hearts (Triangular Trade Trilogy, #1): £13.99
Shakespeare's Sonnets: £20.66
Set Me Free: £17.46
Scott Pilgrim's Precious Little Life (Scott Pilgrim #1): £52.29
Rip it Up and Start Again: £35.02
Our Band Could Be Your Life: Scenes from the American Indie Underground, 1981-1991: £57.25
Olio: £23.88
Mesaerion: The Best Science Fiction Stories 1800-1849: £37.59
Libertarianism for Beginners: £51.33
It's Only the Himalayas: £45.17
```

### Scraping a Wikipedia Page

이제 [Wikipedia 페이지](https://en.wikipedia.org/wiki/Web_scraping)를 スクレイ핑하여 모든 링크에 대한 정보를 추출할 수 있는 スクレイ퍼를 만들어 보겠습니다.

`wiki_scraper.py`라는 새 파일을 생성합니다. 이전과 마찬가지로 필요한 라이브러리를 import하고, GET リクエスト를 수행한 뒤 콘텐츠를 파싱합니다:

```python
import requests
from bs4 import BeautifulSoup
import re

page = requests.get('https://en.wikipedia.org/wiki/Web_scraping')
soup = BeautifulSoup(page.content, 'html.parser')
```

`find_all()` 메서드를 사용해 모든 링크를 찾습니다:

```python
links = soup.find_all("a")
content = str(links)
```

링크 텍스트는 `title` 속성에, URL은 `href` 속성에 나타나므로 `(.*?)` 패턴을 사용해 둘 다 캡처할 수 있습니다. 전체 표현식은 다음과 같습니다:

```regex
<a href="(.*?)" title="(.*?)">.*?</a>
```

세 번째 `.*?`는 `a` 태그의 콘텐츠에는 관심이 없으므로 캡처링 그룹에 포함되지 않습니다.

이전과 같이 `findall()`로 모든 매치를 찾고 결과를 출력합니다:

```python
re_links = r'<a href="(.*?)" title="(.*?)">.*?</a>'

links = re.findall(re_links, content)

for i in links:
    print(f"{i[0]} => {i[1]}")
```

`python wiki_scraper.py`로 실행하면 다음과 같은 출력이 나타납니다:

```
OUTPUT TRUNCATED FOR BREVITY

/wiki/Category:Web_scraping => Category:Web scraping
/wiki/Category:CS1_maint:_multiple_names:_authors_list => Category:CS1 maint: multiple names: authors list
/wiki/Category:CS1_Danish-language_sources_(da) => Category:CS1 Danish-language sources (da)
/wiki/Category:CS1_French-language_sources_(fr) => Category:CS1 French-language sources (fr)
/wiki/Category:Articles_with_short_description => Category:Articles with short description
/wiki/Category:Short_description_matches_Wikidata => Category:Short description matches Wikidata
/wiki/Category:Articles_needing_additional_references_from_April_2023 => Category:Articles needing additional references from April 2023
/wiki/Category:All_articles_needing_additional_references => Category:All articles needing additional references
/wiki/Category:Articles_with_limited_geographic_scope_from_October_2015 => Category:Articles with limited geographic scope from October 2015
/wiki/Category:United_States-centric => Category:United States-centric
/wiki/Category:All_articles_with_unsourced_statements => Category:All articles with unsourced statements
/wiki/Category:Articles_with_unsourced_statements_from_April_2023 => Category:Articles with unsourced statements from April 2023
```

### Scraping a Dynamic Site

이전 예시는 정적 웹 페이지를 다루었습니다. 동적 페이지를 スクレイピング하려면 [Selenium](https://www.selenium.dev/)과 같은 브라우저 자동화 도구가 필요합니다. 다음은 Selenium과 regex를 사용하여 [OpenWeatherMap](https://openweathermap.org/)에서 현재 온도를 추출하는 예시입니다:

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import re

driver = webdriver.Firefox()
driver.get("https://openweathermap.org/city/2643743")
elem = WebDriverWait(driver, 10).until( EC.presence_of_element_located((By.CSS_SELECTOR, ".current-temp")))
content = elem.get_attribute('innerHTML')
re_temp = r'<span .*?>(.*?)</span>'
temp = re.findall(re_temp, content)
print(repr(temp))
driver.close()
```

이 코드는 Selenium을 사용해 Firefox 인스턴스를 실행하고, CSS selector를 사용해 현재 온도가 있는 요소를 선택합니다. 그런 다음 regex `<span .*?>(.*?)</span>`를 사용해 온도를 추출합니다.

이 코드는 Selenium으로 Firefox를 실행하고, CSS selector를 사용해 현재 온도를 포함한 요소를 선택한 다음, regex 패턴 `<span .*?>(.*?)</span>`를 사용하여 온도를 추출합니다.

Selenium으로 동적 웹 페이지를 スクレイピング하는 방법을 시작하는 데 도움이 되는 추가 정보는 [이 튜토리얼](https://brightdata.co.kr/blog/how-tos/using-selenium-for-web-scraping)을 확인하십시오.

## Limitation of Regex for Web Scraping

정규 표현식은 패턴 매칭과 데이터 추출에 강력하지만, Webスクレイピング에는 중요한 한계가 있습니다. regex는 HTML 구조를 이해하지 못한 채 텍스트만을 대상으로 작동하므로, 결과가 HTML의 포맷팅에 크게 의존합니다.

예를 들어 Wikipedia 예시에서 일부 링크는 올바르게 추출되지 않았습니다:

![Links that weren't exctracted correctly](https://github.com/luminati-io/web-scraping-with-regex/blob/main/images/Links-that-werent-exctracted-correctly.png)

Python 코드를 편집하여 `print(content)`를 추가하고 Beautiful Soup가 반환한 HTML 문자열을 출력하면, 문제의 `a`가 다음과 같이 보입니다:

```regex
<a href="#cite_ref-9">^</a>
```

이 태그에는 `title` 속성이 없지만, 우리의 regex 패턴은 `<a href="(.*?)" title="(.*?)">.*?</a>` 구조를 가정합니다. regex는 HTML 요소를 이해하지 못하기 때문에 매칭에 실패하는 대신 `.*?` 패턴이 패턴을 완성하는 무언가를 찾을 때까지 계속 문자를 매칭하며, 종종 여러 태그를 잘못 캡처하게 됩니다.

또한 HTML은 정규 언어가 아니므로, regex만으로 임의의 HTML을 신뢰성 있게 파싱할 수 없습니다. 하지만 regex는 특정 시나리오에서는 유용할 수 있습니다. 제한적이고 알려진 HTML 구조를 다루는 경우, regex는 정보를 효과적으로 추출할 수 있습니다. 그럼에도 더 견고한 접근은 Beautiful Soup 같은 HTML 파서를 사용해 요소를 찾고, 추출된 텍스트를 처리하는 데 regex를 적용하는 것입니다.

아래는 Wikipedia スクレ이퍼의 개선된 버전으로, 초기 추출에는 Beautiful Soup를 사용하고 필터링에는 regex를 사용합니다:

```python
import requests
from bs4 import BeautifulSoup
import re

page = requests.get('https://en.wikipedia.org/wiki/Web_scraping')
soup = BeautifulSoup(page.content, 'html.parser')
links = soup.find_all("a")

for link in links:
    href = link.get('href')
    title = link.get('title')

    if title == None:
        title = link.string

    if title == None:
        continue

    pattern = r"[a-zA-Z0-9]"

    if re.match(pattern, title):
        print(f"{href} => {title}")
```

## Conclusion

정규 표현식은 텍스트 데이터에서 패턴을 찾는 데 유용한 도구입니다. 그러나 Webスクレイピング은 regex의 역량을 넘어서는 다양한 과제를 제시합니다. 스크レイピング을 자주 수행하면 IPアドレス 차단으로 이어질 수 있으며, CAPTCHA가 スクレイ퍼의 기능을 방해할 수 있습니다. Bright Data는 IP 제한을 극복하는 데 도움이 되는 [강력한 プロキシ](https://brightdata.co.kr/proxy-types)를 제공합니다.

지금 무료 체험을 시작하십시오!