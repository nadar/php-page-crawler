# Page Crawler

![Tests](https://github.com/nadar/php-page-crawler/workflows/Tests/badge.svg)
[![Test Coverage](https://api.codeclimate.com/v1/badges/457045a9df14082dcc75/test_coverage)](https://codeclimate.com/github/nadar/php-page-crawler/test_coverage)
[![Maintainability](https://api.codeclimate.com/v1/badges/457045a9df14082dcc75/maintainability)](https://codeclimate.com/github/nadar/php-page-crawler/maintainability)

A highly extendible, dependency free Crawler for HTML, PDFS or any other type of Documents.

**Why another Page Crawler?** Yes, indeed, there are already very good parsers around but this parsers should:

+ Dependency Free - we don't want to use any HTTP client, as much PHP "native" code as possible in order to keep the overhead small
+ Memory Efficent - As memory efficient as possible, less overhead, full code control.
+ Extendible - Attach your own parsers in order to determine how html or any other format is parsed. We provided out of the box support for HTML and PDF.

## Installation

```
composer require nadar/page-crawler
```

## Usage

Create your custom handler, this is the classes which will interact with the crawler in order to store your content somwehere. This function will run after each url is crawled:

```php
class MyCrawlHandler implements \Nadar\PageCrawler\Interfaces\HandlerInterface
{
    public function afterRun(\Nadar\PageCrawler\Result $result)
    {
        echo $result->title;
    }
}
```

The handler class within the full setup:

```php
$crawler = new Crawler('https://luya.io');

// what kind of document types would you like to parse?
$crawler->registerParser(new Nadar\PageCrawler\Parsers\Html);

// register your handler in order to interact with the results, maybe store them in a database?
$crawler->registerHandler(new MyCrawlHandler);

// start the crawling
$crawler->run();
```

## Benchmark

Of course those benchmarks may vary depending on internet connection, bandwidth, servers but we made all the tests under the same circumstances. The memory peak varys strong when using the PDF parsers.

| Index Size     | Concurrent Requests    | Memory Peak     |Time               | Parsers
|-------------- |-------------------    |-----------        |----
| 3785          | 15                    | 18 MB             | 260 Seconds       | Html
| 1509          | 30                    | 97 MB             | 225 Seconds       | Html, PDF
| 1374          | 30                    | 269 MB            | 87 Seconds        | Html, PDF

This is the example benchmark setup:

```php
use Nadar\PageCrawler\Crawler;
use Nadar\PageCrawler\Handlers\DebugHandler;
use Nadar\PageCrawler\Parsers\HtmlParser;
use Nadar\PageCrawler\Parsers\PdfParser;

include 'vendor/autoload.php';

$handler = new DebugHandler();

$crawler = new Crawler('.........YOUR_WEBSITE........');
$crawler->addParser(new HtmlParser);
$crawler->addParser(new PdfParser);
$crawler->addHandler($handler);
$crawler->start();

echo "==================" . PHP_EOL;
echo "URLs: " . ($handler->counter) . PHP_EOL;
echo "time: " . ($handler->elapsedTime()) . PHP_EOL;
echo "peak: " . $handler->memoryPeak() . PHP_EOL;
```