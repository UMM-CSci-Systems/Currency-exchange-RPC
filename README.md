# Remote procedure call (RPC) for Currency Exchange

This lab explores the idea of client-server organization.  In a client-server
configuration one central machine called the **server** acts as a central source
for some resource or service.  Other machines known as **clients** utilize the
resource or service provided by the server.  A good example would be a web-server
providing web-pages to multiple browsers on multiple computers.  

The term **client-server** refers to the configuration of the service provider and
service consumer.  It does not require multiple machines. It is possible for the same
computer to act as server and as client.

This lab illustrates the encapsulation of a
XML-based web service as a simple remote procedure call (RPC), where we wrap
a call to a remote service (a service provides currency exchange rate data)
in a way that allows users to access that service through what appear to be
local function calls.

##Table of contents

-   [RPC encapsulation](#-rpc-encapsulation)
    -   [Reading from a URL](#-reading-from-a-url)
    -   [Parsing XML](#-parsing-xml)

------------------------------------------------------------------------

### <span name="RPC_encapsulation"></span> RPC encapsulation

The starter code and some JUnit tests for this part of the lab are provided
in this repository; you
should fork this
project
and complete your implementation in your group's fork.

Financial programs often need access to financial rates such as stock prices and
exchange rates. There are numerous services out there that provide that
data, often through rich (and complex) APIs. There are also, however,
simple services that provide the data in XML format in response to
standard HTTP requests. [Xavier Media](http://www.xaviermedia.com/), for
example, provides a simple currency exchange rate service that allows
you to specify a date as components of a URL, and generates the exchange
rates for a number of major currency for that date. The URL
<http://api.finance.xaviermedia.com/api/2010/09/09.xml> for example
generates an XML document containing a variety of exchange rates for 9
Sep 2010. See [Xavier's
documentation](http://www.xavierforum.com/viewtopicb2bb.html?f=5&t=10979)
for more info. This is nice if we just want to look up a single date and
read through it by hand, but is somewhat awkward if we want to access
this data programmatically (i.e., as part of a piece of software we're
writing). The goal of this part of the lab is to build a simple RPC
encapsulation of this service, essentially providing a wrapper that
isolates users (programmers in this case) from the details of accessing
and parsing the data. We'll provide two key methods:

```java
public float getExchangeRate(String currencyCode, int year, int month, int day);

public float getExchangeRate(String fromCurrency, String toCurrency, int year, int month, int day);
```

The first provides the exchange on the given date for the given currency
against the base currency (which for Xavier is the Euro). The second
takes a date and two currencies, and returns the exchange rate of the
first vs. the second. Currencies are specified using [ISO 4217 currency
codes](http://en.wikipedia.org/wiki/ISO_4217), and dates are the year
(as a four digit integer), the month as a two digit integer (01=Jan,
12=Dec), and the day of the month as a two digit integer. We've provided
some simple JUnit tests and a stub in the project. The
first four tests all reference static XML files which Nic has provided on
`www.morris.umn.edu`; these are also included in the project in the
`XML_files` directory. The fifth one (which is initially marked with
`@Ignore` so it won't actually run) refers to Xavier's web site. You should
wait until you get the first four to pass before you try the last one as
we don't want to be hammering on Xavier's web site while we're trying to
get our code to work. When you're ready to run that last test just
remove the `@Ignore` line and it will run. There are two major pieces
here that you may have never seen:

-   You'll need to read the result of requesting a URL
-   You'll need to parse an XML document

#### <span name="Reading_from_a_URL"></span> Reading from a URL

This is actually quite easy in Java. This little block of code:

```java
String urlString = "http://www.morris.umn.edu/";
URL url = new URL(urlString);
InputStream xmlStream = url.openStream();
```

will generate an `InputStream` that will provide the (HTML) contents of
the Morris home page. You can then pass that `InputStream` to any other reading
tools like a `BufferedReader` or (or more importantly for this lab) an
XML parser.

#### <span name="Parsing_XML"></span> Parsing XML

There are a ton of Java XML parsing tools out there, including several
included as part of Java's standard libraries. We're not going to provide
a full tutorial here, but there's lots of stuff out there on the
Internet. [This simple
example](https://gist.github.com/NicMcPhee/7131454), for example,
provides a rough overview of what's needed, heavily based
on [this
example](http://www.java-tips.org/java-se-tips/javax.xml.parsers/how-to-read-xml-file-in-java.html).

# Compiling & running the JUnit tests on the command line

If you load this project up in Eclipse, then Eclipse will manage all the weirdness in compiling and running
the tests. If you want use a lighter-weight editor like Atom and compile and run on the command line,
however, you'll need to do some slightly fancy command line stuff to get things to work.

To compile the unit tests you need to go into the `test` directory (but no deeper) and run:

```bash
javac -cp .:/usr/share/java/junit.jar:../src xrate/ExchangeRateTest.java
```

The `-cp` flag sets the _classpath_, which is where Java will look for other classes while it compiles
the tests. This is a colon (`:`) separated list of directories and `jar` files, and in this case includes
the `junit.jar` file and the `src` directory for this project (which is where your RPC code lives).

To run the unit tests you need to again be in the `test` directory and run:

```bash
java -cp .:/usr/share/java/junit.jar:/usr/share/java/hamcrest/core.jar:../src org.junit.runner.JUnitCore xrate.ExchangeRateTest
```

Here again we have to set up the classpath (which needs the additional `hamcrest/core.jar` which is a JUnit 
dependency); we also need to add `../src` to the classpath so that Java can find your implementation of `ExchangeRateReader`.
We then say we're running the `JUnitCore` test runner, and passing it out test class name
(`xrate.ExchangeRateTest`) as an argument telling it where to find the tests. This should run all the
tests, printing out information on how many passed, etc.
