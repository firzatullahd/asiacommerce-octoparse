# AsiaCommerce Web Scraping Docs

    The word `Scrape` and `Extract` is being used interchangeably.

## Requirements

- Octoparse, Octoparse username and Password
- Username, and Password of onlinestore.id hosting.
  To test program locally.
- XAMPP
- Access to remote MySQL (ensure that if you try your program locally, your IP address is added to remote MySQL).

## Introduction

- Scraping with Octoparse is mainly done by 2 template tasks, `Get URL` and `Get Data`, and one Web Scraper program consists of multiple `Get URL` Tasks, and multiple `Get Data` Tasks or just one Task that run few times.
- Each Task has specific `Task ID`, that is used to `Start Task`, `Check Task Status`, `Add Task URL`, `Empty Task URL`, `Export Task Data` & `Clear Task Data` through [Octoparse Advanced API](http://advancedapi.octoparse.com/help).
- Each `Task ID` is stored inside `$getURLTaskId` variable for `Get URL` Task, and inside `$getDataTaskId` variable for `Get Data` Task.

# How To Create New Web Scraper.

- Make sure that the products have 3 level of categories, because it is needed for ALI
for example : `Toys & Games › Learning & Education › Electronic Learning Products`.
1. Login to Octoparse.
2. Click `+ New` and `Create a New Group`, name it according to the website you are going to scrape.
3. Click Dashboard, select one of Rakuten `Get URL Task`, e.g., `Rakuten - Beauty - Get URL` and duplicate it.
4. Click Dashboard, select one of Rakuten `Get Data Task`, i.e., `Rakuten - All Product - Extract` and duplicate it.
5. Move both recently duplicated tasks into new group, change each task's name to `<website name> - Get URL` and `<website name> - Get Data. All Product - Extract`, or anything else that would help you differentitate each task.
6. Open duplicated `Get URL Task`, click action setting inside Loop Item/Loop URL, click Loop Item, delete all URLs, and input URL of new website category page(this page should contains product list).
7. Delete Loop Item that contains extract data.
8. Next step is to extract URL of each product. I suggest to watch [this tutorial](https://www.youtube.com/watch?v=cKpTLOaiU6Y). Make sure the captured field name is `Info_URL` (this step is important), and add data field of `Current date & time`(this step is also important to prevent some error in octoparse), you may run `cloud extract` by clicking the play button which says `Run in the Cloud` to see if the task run properly.
9. Open duplicated `Get Data Task`, copy one/more extracted URL, `Info_URL` field from `Get URL Task` and paste it into Loop URLs, Loop Item.
10. Inside Workflow, delete everything, under `Go to Web Page`/`Open Website`.
11. Next step is to extract necessary info from each product. watch [this tutorial](https://www.youtube.com/watch?v=2IVEL-uE4RY). Make sure to add data field of `Current date & time` twice, so there will be `Current_Time` and `Current_Time1` it will be used for different purpose. click the 3 dots of `Current_Time`, and select `Clean data`, add step of Reformat extracted date/time select `yyyy/MM/dd`, you may run `cloud extract` by clicking the play button which says `Run in the Cloud` to see if the task run properly.
12. If both tasks run properly, Create new table in DB, use `WebsiteName_Products` as name. design the database according to captured data field of `Get Data Task`, e.g., title, price, Page_URL, etc. Make sure to use unique field, e.g, title or Page_URL as PRIMARY KEY. Make sure to use the name `time` for `Current_Time` field.
13. Download `extractRakuten.php` from file manager (public_html/ali_plugin/...) in the server, and change its name.
14. Open Octoparse Dashboard, locate your newly duplicated tasks. click more, and under Cloud runs, click API, copy each task Task ID, and paste it into `extractRakuten.php`, `$getURLTaskId` and `$getDataTaskId` respectively, replacing the previous Task ID of Rakuten Tasks.
15. Open `extractRakuten.php`, inside function `exportToDB` modify according to captured field and newly created DB. Make sure to also change the query inside variable `$query`.
16. Run your PHP program locally, make sure to turn on Apache module in XAMPP. There will always be error, so to troubleshoot and make use of echo and var_dump.
17. If Everything run perfectly, scraped data have been exported to DB. you need to delete all echo, var_dump.
18. Upload your php file into the same directory of `extractRakuten.php`.
19. Set Cron jobs by copy paste one of Current Cron Jobs Command and change it according to your php file name, i.e, `/usr/local/bin/ea-php72 /home/..../public_html/.../yourfilename.php`, set the common settings to Once Per Week [This website could probaby help how](https://crontab.guru/). Make sure to set the schedule different with existing Web Scraper, to prevent them from running at the same time, this will reduce the effectiveness of Octoparse, and some page might fail to load.

## Web Scraping Flow

1. Manually copy category page's URL (this category page should contain list of products)
2. manually paste it into `Get URL` Task as Input URL(s).

3. Run Cronjob to start Task `Get URL`, in order to extract Product URLs.
4. Continuously check `Get URL` Task status.
5. If task is `completed` or `stopped`, then Product URLs exracted is exported into a variable,
6. If there are more `Get URL` Task then proceed through point 3-5, once all `Get URL` Task extracted data is exported. It will be added to `Get Data` Task using Octoparse Advanced API.
7. Start `Get Data` Task to extract Product Data.
8. Continuously check `Get Data` Task status.
9. If task is `completed` or `stopped`, then Product Data extracted will be exported to DB `onlinestore_testing`.
10. If Product URLs added into `Get Data` Task is more than 10.000 URLs, point 7-8 will be repeated.


## Deployed Web Scrapers

All 3 Web Scraper is automatically running by using Cronjob.

- [Rakuten](https://sm.rakuten.co.jp/)
- [BigC](https://www.bigc.co.th/)
- [Amazon](https://www.amazon.com/)


## Couple things to note

- Every Website might behave differently, understanding [Xpath](https://www.octoparse.com/tutorial-7/xpath) will certainly help to build scraping task in Octoparse.
- There are common cases such as, [scraping/extract completed but no data is extracted](https://www.octoparse.com/tutorial-7/octoparse-stops-and-no-data-extracted), or [data is extracted but many lines of data are blanks except page_url](https://www.octoparse.com/tutorial-7/get-no-data-even-i-do-see-it-in-workflow). I just want to say that it is unavoidable, just run the scraper again.
- As I've mention previously, that every website behave differently, Amazon is a special one so far. It has bot detector that might resulted in we got many lines of data that say `Sorry Something went wrong` or just `Amazon.com`. Make Sure that [Octoparse Anti-Blocking System is enabled](https://helpcenter.octoparse.com/hc/en-us/articles/360031224172-Octoparse-Anti-Blocking-Settings), but even so, it will just reduces the possibility of error and not entirely get rid of it.
- Octoparse is not perfect. We could expect it to scrape 100 product data. Sometimes it scrapes all 100 data, other times it scrapes less of 100, some other time it scrapes nothing.
- Octoparse task only accept 10.000 URLs as input. So, each `Get Data` Task might run few times.

