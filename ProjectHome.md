SitemapGen4j is a library to generate XML sitemaps in Java.

<h2>What's an XML sitemap?</h2>

Quoting from <a href='http://sitemaps.org/index.php'>sitemaps.org</a>:

<blockquote><p>Sitemaps are an easy way for webmasters to inform search engines about pages on their sites that are available for crawling. In its simplest form, a Sitemap is an XML file that lists URLs for a site along with additional metadata about each URL (when it was last updated, how often it usually changes, and how important it is, relative to other URLs in the site) so that search engines can more intelligently crawl the site.</p>

<p>Web crawlers usually discover pages from links within the site and from other sites. Sitemaps supplement this data to allow crawlers that support Sitemaps to pick up all URLs in the Sitemap and learn about those URLs using the associated metadata. Using the Sitemap protocol does not guarantee that web pages are included in search engines, but provides hints for web crawlers to do a better job of crawling your site.</p>

<p>Sitemap 0.90 is offered under the terms of the Attribution-ShareAlike Creative Commons License and has wide adoption, including support from Google, Yahoo!, and Microsoft.</p>
</blockquote>

<h2>Getting started</h2>

<p>The easiest way to get started is to just use the WebSitemapGenerator class, like this:<br>
<br>
<pre><code>WebSitemapGenerator wsg = new WebSitemapGenerator("http://www.example.com", myDir);<br>
wsg.addUrl("http://www.example.com/index.html"); // repeat multiple times<br>
wsg.write();<br>
</code></pre>

<h2>Configuring options</h2>

But there are a lot of nifty options available for URLs and for the generator as a whole.  To configure the generator, use a builder:<br>
<br>
<pre><code>WebSitemapGenerator wsg = WebSitemapGenerator.builder("http://www.example.com", myDir)<br>
    .gzip(true).build(); // enable gzipped output<br>
wsg.addUrl("http://www.example.com/index.html");<br>
wsg.write();<br>
</code></pre>

To configure the URLs, construct a WebSitemapUrl with WebSitemapUrl.Options.<br>
<br>
<pre><code>WebSitemapGenerator wsg = new WebSitemapGenerator("http://www.example.com", myDir);<br>
WebSitemapUrl url = new WebSitemapUrl.Options("http://www.example.com/index.html")<br>
    .lastMod(new Date()).priority(1.0).changeFreq(ChangeFreq.HOURLY).build();<br>
// this will configure the URL with lastmod=now, priority=1.0, changefreq=hourly <br>
wsg.addUrl(url);<br>
wsg.write();<br>
</code></pre>

<h2>Configuring the date format</h2>

One important configuration option for the sitemap generator is the date format.  The <a href='http://www.w3.org/TR/NOTE-datetime'>W3C datetime standard</a> allows you to choose the precision of your datetime (anything from just specifying the year like "1997" to specifying the fraction of the second like "1997-07-16T19:20:30.45+01:00"); if you don't specify one, we'll try to guess which one you want, and we'll use the default timezone of the local machine, which might not be what you prefer.<br>
<br>
<pre><code><br>
// Use DAY pattern (2009-02-07), Greenwich Mean Time timezone<br>
W3CDateFormat dateFormat = new W3CDateFormat(Pattern.DAY); <br>
dateFormat.setTimeZone(TimeZone.getTimeZone("GMT"));<br>
WebSitemapGenerator wsg = WebSitemapGenerator.builder("http://www.example.com", myDir)<br>
    .dateFormat(dateFormat).build(); // actually use the configured dateFormat<br>
wsg.addUrl("http://www.example.com/index.html");<br>
wsg.write();<br>
</code></pre>

<h2>Lots of URLs: a sitemap index file</h2>

One sitemap can contain a maximum of 50,000 URLs.  (Some sitemaps, like Google News sitemaps, can contain only 1,000 URLs.) If you need to put more URLs than that in a sitemap, you'll have to use a sitemap index file.  Fortunately,  WebSitemapGenerator can manage the whole thing for you.<br>
<br>
<pre><code>WebSitemapGenerator wsg = new WebSitemapGenerator("http://www.example.com", myDir);<br>
for (int i = 0; i &amp;lt; 60000; i++) wsg.addUrl("http://www.example.com/doc"+i+".html");<br>
wsg.write();<br>
wsg.writeSitemapsWithIndex(); // generate the sitemap_index.xml<br>
<br>
</code></pre>

<p>That will generate two sitemaps for 60K URLs: sitemap1.xml (with 50K urls) and sitemap2.xml (with the remaining 10K), and then generate a sitemap_index.xml file describing the two.</p>

<p>It's also possible to carefully organize your sub-sitemaps.  For example, it's recommended to group URLs with the same changeFreq together (have one sitemap for changeFreq "daily" and another for changeFreq "yearly"), so you can modify the lastMod of the daily sitemap without modifying the lastMod of the yearly sitemap.  To do that, just construct your sitemaps one at a time using  the WebSitemapGenerator, then use the SitemapIndexGenerator to create a single index for all of them.</p>

<pre><code>WebSitemapGenerator wsg;<br>
// generate foo sitemap<br>
wsg = WebSitemapGenerator.builder("http://www.example.com", myDir)<br>
    .fileNamePrefix("foo").build();<br>
for (int i = 0; i &amp;lt; 5; i++) wsg.addUrl("http://www.example.com/foo"+i+".html");<br>
wsg.write();<br>
// generate bar sitemap<br>
wsg = WebSitemapGenerator.builder("http://www.example.com", myDir)<br>
    .fileNamePrefix("bar").build();<br>
for (int i = 0; i &amp;lt; 5; i++) wsg.addUrl("http://www.example.com/bar"+i+".html");<br>
wsg.write();<br>
// generate sitemap index for foo + bar <br>
SitemapIndexGenerator sig = new SitemapIndexGenerator("http://www.example.com", myFile);<br>
sig.addUrl("http://www.example.com/foo.xml");<br>
sig.addUrl("http://www.example.com/bar.xml");<br>
sig.write();<br>
</code></pre>

<p>You could also use the SitemapIndexGenerator to incorporate sitemaps generated by other tools.  For example, you might use Google's official Python sitemap generator to generate some sitemaps, and use WebSitemapGenerator to generate some sitemaps, and use SitemapIndexGenerator to make an index of all of them.</p>

<h2>Validate your sitemaps</h2>

<p>SitemapGen4j can also validate your sitemaps using the official XML Schema Definition (XSD).  If you used SitemapGen4j to make the sitemaps, you shouldn't need to do this unless there's a bug in our code.  But you can use it to validate sitemaps generated by other tools, and it provides an extra level of safety.</p>

<p>It's easy to configure the WebSitemapGenerator to automatically validate your sitemaps right after you write them (but this does slow things down, naturally).</p>

<pre><code>WebSitemapGenerator wsg = WebSitemapGenerator.builder("http://www.example.com", myDir)<br>
    .autoValidate(true).build(); // validate the sitemap after writing<br>
wsg.addUrl("http://www.example.com/index.html");<br>
wsg.write();<br>
</code></pre>

<p>You can also use the SitemapValidator directly to manage sitemaps.  It has two methods: validateWebSitemap(File f) and validateSitemapIndex(File f).</p>

<h2>Google-specific sitemaps</h2>

<p>Google can understand a wide variety of custom sitemap formats that they made up, including a Mobile sitemaps, Geo sitemaps, Code sitemaps (for Google Code search), Google News sitemaps, and Video sitemaps.  SitemapGen4j can generate any/all of these different types of sitemaps.</p>

<p>To generate a special type of sitemap, just use GoogleMobileSitemapGenerator, GoogleGeoSitemapGenerator, GoogleCodeSitemapGenerator, GoogleCodeSitemapGenerator, GoogleNewsSitemapGenerator, or GoogleVideoSitemapGenerator instead of WebSitemapGenerator.</p>

<p>You can't mix-and-match regular URLs with Google-specific sitemaps, so you'll also have to use a GoogleMobileSitemapUrl, GoogleGeoSitemapUrl, GoogleCodeSitemapUrl, GoogleNewsSitemapUrl, or GoogleVideoSitemapUrl instead of a WebSitemapUrl.  Each of them has unique configurable options not available to regular web URLs.</p>