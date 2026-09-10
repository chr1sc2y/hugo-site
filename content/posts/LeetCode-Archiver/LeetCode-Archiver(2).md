---
title: "LeetCode Archiver (2): Retrieving Problem Data"
date: 2018-12-21T15:06:01+11:00
draft: false
categories: ["Python"]
description: "A translated technical note on LeetCode Archiver (2): Retrieving Problem Data, preserving the examples and context of the original article."
---
## Create a crawler

> Originally published in Chinese on 2018-12-21; this English edition preserves the original scope and technical context.

After creating a project, open it with PyCharm or any other IDE. Navigate to the project folder and use the command ```genspider``` to create a spider:
```
cd scrapy_project
scrapy genspider QuestionSetSpider leetcode.com
```
Among them, QuestionSetSpider is the name of the crawler, and leetcode.com is the domain name of the website we intend to crawl.

After creating the new crawler, you can see that a new file named QuestionSetSpider.py has been added to the spiders folder of the project. This is the crawler file we just created. This crawler file will automatically generate the following code
```
# LeetCode Archiver (2): Retrieving Problem Data
import scrapy

class QuestionSetSpider(scrapy.Spider):
    name = 'QuestionSetSpider'
    allowed_domains = ['leetcode.com']
    start_urls = ['http://leetcode.com/']

    def parse(self, response):
        pass

```
- The QuestionSetSpider class inherits from scrapy.Spider, which is the base class of all crawlers in the scrapy framework;
- The self.name attribute is the name of the crawler. The current crawler can be obtained through this attribute outside the crawler file;
- self.allowed_domains is a list of domain names that the current crawler file can access. If a URL other than the domain name is entered when crawling the page, an error will be thrown;
- self.start_urls is a list of URLs. The start_requests function is defined in the base class. It will traverse self.start_urls and call scrapy.Request(url, dont_filter=True) for each URL. In order to meet the needs of crawling questions, we need to rewrite the self.start_urls function.

## Get question details

### Analysis

LeetCode uses GraphQL for data query and transmission. Most pages are dynamic pages generated through JS rendering, so tags cannot be obtained directly from the page. Even if you use a library that provides JavaScript rendering services (such as Splash), you cannot obtain all the data, so you can only obtain the data by sending a request.

In order to crawl the detailed information of the topic, we first need to enter the link corresponding to each topic from the topic list.

First open leetcode's [problem](https://leetcode.com/problemset/all/) list, press F12 to open Chrome's developer tools, enter the Network tab bar, check Preserve log, and refresh the page.

![1](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Use-Scrapy-to-Crawl-LeetCode/1.png)

As you can see, the webpage sends a GET type Request named "all/" to https://leetcode.com/api/problems/all/. This is a request to obtain all question links and related information. If the Toggle JavaScript plug-in has been installed at this time, we can directly right-click "Open in new tab" to view the Response returned by the request.

![2](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Use-Scrapy-to-Crawl-LeetCode/2.png)

A more convenient method is to use postman to send the same Request to the server and save it, so that if we need to view the corresponding Response next time, we do not need to use the developer tools.

![3](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Use-Scrapy-to-Crawl-LeetCode/3.png)

The returned Response is a json object, in which the value corresponding to the "stat_status_pairs" key is a list containing all question information, and ["stat"]["question__title_slug"] in the list is the page where the question is located. Take the Largest Perimeter Triangle as an example. After splicing its title_slug to https://leetcode.com/problems/, enter the page https://leetcode.com/problems/largest-perimeter-triangle/. Similarly, open the developer tools and refresh the page. You can see that the server has returned many items of graphql query data. By looking at the Request Payload, you can find one item whose operationName is "questionData". This is the detailed information of the current question.

![4](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Use-Scrapy-to-Crawl-LeetCode/4.png)

Copy and paste the Payload into the Postman Body, set the Content-Type in the Headers to application/json, and send the request. You can see that a json object is returned, which contains all the information corresponding to the question.

![5](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Use-Scrapy-to-Crawl-LeetCode/5.png)

Next we can process the information on this topic.

### Implementation

In order to obtain the json object of the question list, we need to rewrite the start_requests function first.
```
def start_requests(self):
        self.Login() # User login, will be used later
        questionset_url = "https://leetcode.com/api/problems/all/"
        yield scrapy.Request(url=questionset_url, callback=self.ParseQuestionSet)
```
Request is a class object of scrapy. Its function is similar to the get function in the requests library. It allows the Downloader in the scrapy framework to send a get request to the URL and hand the obtained response to the callback function in the specified crawler file for corresponding processing. Its constructor is as follows
```
class Request(object_ref):

    def __init__(self, url, callback=None, method='GET', headers=None, body=None, cookies=None, meta=None, encoding='utf-8', priority=0, dont_filter=False, errback=None, flags=None):
    ...
```
After obtaining the json object, you can get the title_slug of the question by traversing the list corresponding to the "stat_status_pairs" key and taking out the value of ["stat"]["question__title_slug"]. At this point, we no longer need to open the question-related page, and can directly send a request to query detailed information to GraphQL.

We can get the code related to sending the request directly from postman. Because the title_slug of each title is different, we can change the field after the titleSlug in the payload to a unique string that will not be repeated. After each time a new title_slug is obtained, replace it with the replace function, send a new request, and then replace it back with a unique string.

![6](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/Use-Scrapy-to-Crawl-LeetCode/6.png)

After preparing the Payload and Headers, we can use FormRequest to send a POST request to query data from GraphQL. FormRequest is a class object of scrapy. Its function is similar to the post function in the requests library. It allows the Downloader in the scrapy framework to send a post request to the URL and hands the obtained response to the callback function in the specified crawler file for corresponding processing. Here, after sending the POST request, the response is handed over to the ParseQuestionData function for processing.
```
    question_payload = "{\n    \"operationName\": \"questionData\",\n    \"variables\": {\n        \"titleSlug\": \"QuestionName\"\n    },\n    \"query\": \"query questionData($titleSlug: String!) {\\n  question(titleSlug: $titleSlug) {\\n    questionId\\n    questionFrontendId\\n    boundTopicId\\n    title\\n    titleSlug\\n    content\\n    translatedTitle\\n    translatedContent\\n    isPaidOnly\\n    difficulty\\n    likes\\n    dislikes\\n    isLiked\\n    similarQuestions\\n    contributors {\\n      username\\n      profileUrl\\n      avatarUrl\\n      __typename\\n    }\\n    langToValidPlayground\\n    topicTags {\\n      name\\n      slug\\n      translatedName\\n      __typename\\n    }\\n    companyTagStats\\n    codeSnippets {\\n      lang\\n      langSlug\\n      code\\n      __typename\\n    }\\n    stats\\n    hints\\n    solution {\\n      id\\n      canSeeDetail\\n      __typename\\n    }\\n    status\\n    sampleTestCase\\n    metaData\\n    judgerAvailable\\n    judgeType\\n    mysqlSchemas\\n    enableRunCode\\n    enableTestMode\\n    envInfo\\n    __typename\\n  }\\n}\\n\"\n}\n"

    graphql_url = "https://leetcode.com/graphql"

    def ParseQuestionSet(self, response):
        headers = {
            "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36'",
            "content-type": "application/json"  # necessary
        }
        questionSet = json.loads(response.text)
        questionSet = questionSet["stat_status_pairs"]
        for question in questionSet:
            title_slug = question["stat"]["question__title_slug"]
            self.question_payload = self.question_payload.replace("QuestionName", title_slug)
            yield scrapy.FormRequest(url=self.graphql_url, callback=self.ParseQuestionData,
                                     headers=headers, body=self.question_payload)
            self.question_payload = self.question_payload.replace(title_slug, "QuestionName")
```
Now that the data has been obtained, we need to define a class in the items.py file to store the detailed information of the item. The classes in the items.py file inherit from the scrapy.Item class, which is a unified data structure provided to the component Item Pipeline in the scrapy framework for processing.
```
import scrapy

class QuestionDataItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    id = scrapy.Field()
    title = scrapy.Field()
    content = scrapy.Field()
    submission_list = scrapy.Field()
    topics = scrapy.Field()
    difficulty = scrapy.Field()
    ac_rate = scrapy.Field()
    likes = scrapy.Field()
    dislikes = scrapy.Field()
    slug = scrapy.Field()
```
After defining the QuestionDataItem class, you can enter the ParseQuestionData function to start extracting the detailed information of the question. We can extract the id, title, content, topics, difficulty and other information of the question according to the needs, use a QuestionDataItem object to store these data, and then perform the yield questionDataItem operation and hand this object to the Item Pipeline for processing.
```
    def ParseQuestionData(self, response):
        questionData = json.loads(response.text)["data"]["question"]
        questionDataItem = QuestionDataItem()
        questionDataItem["id"] = questionData["questionFrontendId"]
        questionDataItem["title"] = questionData["title"]
        questionDataItem["content"] = questionData["content"]
        topics = []
        for topic in questionData["topicTags"]:
            topics.append(topic["name"])
        if len(topics) == 0:
            topics.append("None")
        questionDataItem["topics"] = topics
        questionDataItem["difficulty"] = questionData["difficulty"]
        stats = json.loads(questionData["stats"])
        questionDataItem["ac_rate"] = stats["acRate"]
        questionDataItem["likes"] = questionData["likes"]
        questionDataItem["dislikes"] = questionData["dislikes"]
        questionDataItem["slug"] = questionData["titleSlug"]
        submission_list = self.GetSubmissionList(questionDataItem["slug"])
        questionDataItem["submission_list"] = submission_list

        yield questionDataItem
```
At this point, the crawling of question information is completed.

## References

<a href="https://scrapy-chs.readthedocs.io/zh_CN/0.24/" target="_blank">Scrapy official documentation</a>

<a href="https://learning.getpostman.com/docs/postman/launching_postman/installation_and_updates/">Postman official documentation</a>
