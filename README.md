# AsiaCommerce Web Scraping Docs

    Scrape and Extract is being used interchangeably.

## Requirements

- Octoparse, Octoparse username and Password
- Username, and Password of onlinestore.id hosting.
  To test program locally.
- XAMPP
- Access to remote MySQL (ensure that if you try your program locally, your IP address is added to remote MySQL).

## Deployed Web Scrapers

All 3 Web Scraper is automatically running every Sunday at `00.00`, by using Cronjob.

- [Rakuten](https://sm.rakuten.co.jp/) (Can be run by visiting this URL on browser `https://onlinestore.id/ali_plugin/cron/extractRakuten.php` )
- [BigC](https://www.bigc.co.th/) (Can be run by visiting this URL on browser `https://onlinestore.id/ali_plugin/cron/extractBigC.php` )
- [Amazon](https://www.amazon.com/) (Can be run by visiting this URL on browser `https://onlinestore.id/ali_plugin/cron/extractAmazon.php` )

## Couple things to note

- Every Website might behave differently, understanding [Xpath](https://www.octoparse.com/tutorial-7/xpath) will certainly help to build scraping task in Octoparse.
- There are common cases such as, [scraping/extract completed but no data is extracted](https://www.octoparse.com/tutorial-7/octoparse-stops-and-no-data-extracted), or [data is extracted but many lines of data are blanks except page_url](https://www.octoparse.com/tutorial-7/get-no-data-even-i-do-see-it-in-workflow). I just want to say that it is unavoidable, just run the scraper again.
- As I've mention previously, that every website behave differently, Amazon is a special one so far. It has bot detector that might resulted in we got many lines of data that say `Sorry Something went wrong` or just `Amazon.com`. Make Sure that [Octoparse Anti-Blocking System is enabled](https://helpcenter.octoparse.com/hc/en-us/articles/360031224172-Octoparse-Anti-Blocking-Settings), but even so, it will just reduces the possibility of error and not entirely get rid of it.
- Octoparse is not perfect. We could expect it to scrape 100 product data. Sometimes it scrapes all 100 data, other times it scrapes less of 100, some other time it scrapes nothing.
- Octoparse task only accept 10.000 URLs as input. So, each `Get Data` Task might run few times.

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

## Introduction

- Scraping with Octoparse is mainly done by 2 template tasks, `Get URL` and `Get Data`, and one Web Scraper program consists of multiple `Get URL` Tasks, and multiple `Get Data` Tasks or just one Task that run few times.
- Each Task has specific `Task ID`, that is used to `Start Task`, `Check Task Status`, `Add Task URL`, `Empty Task URL`, `Export Task Data` & `Clear Task Data` through [Octoparse Advanced API](http://advancedapi.octoparse.com/help).
- Each `Task ID` is stored inside `$getURLTaskId` variable for `Get URL` Task, and inside `$getDataTaskId` variable for `Get Data` Task.

## How to modify existing scraper

The Web Scraping program is designed to scrape URL by Categories, so if there are any modification whether to add, delete, or change scraped products. We first need to add/delete a category URL.
for example:

- `https://sm.rakuten.co.jp/search/200600?l-id=_leftnavi_200600&sort=1`
- `https://www.bigc.co.th/home-appliances-electronic-products.html`
- `https://www.amazon.com/s?i=toys-and-games&bbn=166269011&rh=n%3A165793011%2Cn%3A166269011%2Cn%3A166275011%2Cp_n_shipping_option-bin%3A3242350011&dc&fst=as%3Aoff&pf_rd_i=16225015011&pf_rd_m=ATVPDKIKX0DER&pf_rd_p=4a512918-34fa-4e04-a696-c21964050204&pf_rd_r=7XJTTQWYVK6HHV0AY0HR&pf_rd_s=merchandised-search-6&pf_rd_t=101&qid=1605759463&rnid=166269011&ref=sr_nr_n_3`

Make sure that the products have 3 level of categories, because it is needed for ALI
for example : `Toys & Games › Learning & Education › Electronic Learning Products`

- Delete

In order to delete a certain category, simply delete the task and delete the `Task ID` inside `$getURLTaskId` variable.

- Add

In order to add a category, simply copy another `Get URL` Task, add the category URL to the task, copy its `Task ID` and add into `$getURLTaskId` variable.

## How to create a new web scaper

- Create 2 new tasks, one `Get URL` to scrape product details URL from category page, and one `Get Data` to scrape data from product details page.
- Copy the php file of other web scraper inside File Manager such as `extractRakuten.php`.
- Delete all `Task ID`s inside variables `$getURLTaskId` and `$getDataTaskId`, and add in your newly created `Task ID`.
- Create and design new Table inside `onlinestore_testing` DB according to the new website.
- Modify according to how the website behaves, I suggest to try it locally first, because you will have the power of echo & var_dump, while on the server it will get `504 Gateway Timeout Error`. Good luck!
- Once it runs perfectly, delete all echo, var_dump, and change its connection DB to localhost, and just upload your php file into the server.
- Run automatically with Cronjob.

## Things to optimize

- There might be many variable with inconsistent names.
- The Speed to scrape data is quite slow especially for Amazon. There might another way of doing it quickly, few ideas:
  - Modify the task, make use of branch judgement, instead of nested loop.
  - Split the scraped URL exported from `Get URL` Tasks into multiple `Get Data` Tasks and run it simultaneously.
