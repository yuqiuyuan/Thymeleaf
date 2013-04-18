
Thymeleaf Testing Library
=========================

-------------------------------------------------------------

Status
------

This is an auxiliary testing library, not directly a part of the Thymeleaf core but part of the project, developed and supported by the [Thymeleaf Team](http://www.thymeleaf.org/team.html).

Current versions: 

  * **Version 2.0.0** - for Thymeleaf 2.0 (requires 2.0.16+) 


License
-------

This software is licensed under the [Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0.html).


Requirements
------------

  *   Thymeleaf **2.0.16+**
  *   Attoparser **1.2+**


Maven info
----------

  *   groupId: `org.thymeleaf`
  *   artifactId: `thymeleaf-testing`


Features
--------

  *   Works as an independent library, callable from multiple testing frameworks like e.g. JUnit.
  *   Tests only the view layer: template processing and its result.
  *   Includes benchmarking facilities: all tests are timed, times are aggregated.
  *   Highly extensible and configurable
  *   Versatile testing structures: test sequences, iteration, concurrent execution.
  *   Based on interfaces, out-of-the-box *standard test resolution* allows:
      * Easy specification of tests as simple text files, sequences as folders.
	  * Advanced test configuration.
	  * Test inheritance.
	  * *Lenient result matching*, ignoring excess unneeded whitespace etc.

------------------------------------------------------------------------------


## Usage ##

The testing framework can be used with just two lines of code:

```java
final TestExecutor executor = new TestExecutor();
executor.execute("test");
```

Note how here we are only specifying the name of the *testable* to be resolved: `"test"` (more on testables later). But anyway this is only two lines, and therefore we are accepting some defaults, namely:

   * Dialects. By default only the *Standard Dialect* will be enabled.
   * Messages. By default no internationalization messages will be available.
   * Resolvers. By default the *standard test resolution* mechanism will be used (more on it later).
   * Reporters. By default a console reporter will be used.

Let's see the whole `TestExecutor` configuration process:

```java
final List<IDialect> dialects = ...
final Map<Locale,Properties> messages = ...
final ITestableResolver resolver = ...
final ITestReporter reporter = ...

final TestExecutor executor = new TestExecutor();
executor.setDialects(dialects);
executor.setMessages(messages);
executor.setTestableResolver(resolver);
executor.setReporter(reporter);
executor.execute("test");
```

The meaning and working of the *dialects* an *messages* properties is pretty obvious and straightforward. As for the *resolvers* and *reporters*, we will see more on them in next sections.

## API ##

The Thymeleaf testing framework is based on interfaces, and therefore defines an API that specifies the diverse structures involved in the testing process:

Test structures at the `org.thymeleaf.testing.templateengine.testable` package:

| Interface                  | Description |Base Impl| Default Impl|
|----------------------------|-------------|------------------|-------------|
|`ITestable`                 | Implemented by objects designed for being *tested*, be it a simple test or an aggregating structure of any kind. Every other interface in this package extends this one. | `AbstractTestable` | |
|`ITest`                 | Represents tests, the basic unit for testing and simplest `ITestable` implementation. Tests are in charge not only of containing test data, but also of evaluating/checking test results. | `AbstractTest` | `Test` |
|`ITestSequence`                 | Represents sequences of tests, test structures or any combination of theses (sequences of objects implementing the `ITestable` interface). |  | `TestSequence` |
|`ITestIterator`                 | Represents objects capable of iterating (executing a number of times) other test structures. |  | `TestIterator` |
|`ITestParallelizer`                 | Represents objects capable of using several threads for executing the same test structure in each thread, concurrently. |  | `TestParallelizer` |
|`ITestResult`                 | Represents the results of executing a test and evaluating its results. |  | `TestResult` |


Interfaces at the `org.thymeleaf.testing.templateengine.resolver` package:

| Interface                  | Description |
|----------------------------|-------------|
|`ITestableResolver`                 | Implemented by objects in charge of *resolving testables*, this is, of creating the `ITestable` objects and structures that will be executed. A *standard test resolution* implementation is provided out of the box that builds these testable structures from text files and their containing folders in disk. |


Interfaces at the `org.thymeleaf.testing.templateengine.report` package:

| Interface                  | Description |
|----------------------------|-------------|
|`ITestReporter`                 | Implemented by objects in charge of reporting the results of executing tests, sequences, etc. along with their associated execution times. |


In addition to these interfaces, this testing API also includes the `org.thymeleaf.testing.templateengine.engine.TestExecutor` class, in charge of executing the test structures.


## Test Reporters ##

Test Reporters implement the `org.thymeleaf.testing.templateengine.report.ITestReporter` interface and allow the engine to report when a test has been executed, the execution result, and also the execution time (aggregated in the case of a structure).

Out of the box, thymeleaf-testing provides two implementations at the `org.thymeleaf.testing.templateengine.report` package:
   * `AbstractTextualTestReporter`, an abstract text-based implementation suitable for easily creating reporters that output text.
   * `ConsoleTestReporter`, extending the former, which writes these *text report items* to the console.

It's easy to create new reporters that could write test results to different formats like CSV, Excel, etc. or even write results to a database.


## Testable Resolvers ##

Standard test resolution is provided by means of two implementations of the `org.thymeleaf.testing.templateengine.resolver.ITestableResolver` interface, both living at the `org.thymeleaf.testing.templateengine.resolver` package. They are basically two flavors of the same resolution mechanism:

   * `StandardClassPathTestableResolver` for resolving tests from the classpath.
   * `StandardFileTestableResolver` for resolving tests from anywhere in the file system.


### The Standard Resolution mechanism ###

The standard test resolution mechanism works like this:

   * Tests are specified in text files, following a specific directive-based format.
   * Folders can be used for grouping tests into sequences, iterators or parallelizers.
   * Test ordering and sequencing can be configured through the use of *index files*.

Let's see each topic separately.


#### Test file format ####

A test file `simple.test` can look like this:

```
%CONTEXT
onevar = Goodbye!
%TEMPLATE_MODE HTML5
%INPUT
<!DOCTYPE html>
<html>
  <body>
      <h1 th:text="${onevar}">Hello!</h1>
  </body>
</html>
%OUTPUT 
<!DOCTYPE html>
<html>
  <body>
      <h1>Goodbye!</h1>
  </body>
</html>
```

We can see there that tests are configured by means of *directives*, and that this directives are specified in the form of `%NAME`. The available directives are:

*Test Configuration:*

| Name                       | Description |
|----------------------------|-------------|
|`%NAME`                     | Name of the test, in order to make it identifiable in reports/logs. *Optional*. |
|`%CONTEXT`                  | Context variables to be made available to the tested template. These variables can be specified in the form of *properties* (like Java `.properties` files), and property values can optionally be OGNL expressions when enclosed in `${...}`.<br /> Also, a special property called `locale` can be specified in order to configure the locale to be used for template execution.<br />Context is *optional*. |

*Test input:*

| Name                       | Description |
|----------------------------|-------------|
|`%INPUT`                    | Test input, in the form of an HTML template or fragment. This parameter is *required*. |
|`%INPUT[qualif]`              | Additional inputs can be specified by adding a *qualifier* to its name. These additional inputs can be used as external template fragments in `th:include`, `th:substituteby`, etc. |
|`%FRAGMENT`                 | Fragment specification (in the same format as used in `th:include` attributes) to be applied on the test input before processing. *Optional*. |
|`%TEMPLATE_MODE`            | Template mode to be used: `HTML5`, `XHTML`, etc. |
|`%CACHE`                    | Whether template cache should be `on` or `off`. If cache is *on*, the input for this test will be parsed only the first time it is processed.|

*Test expected output:*

| Name                       | Description |
|----------------------------|-------------|
|`%OUTPUT`                   | Test output to be expected, if we expect template execution to finish successfully. Either this or the `%EXCEPTION` directive must be specified. |
|`%EXACT_MATCH`              | Whether *exact matching* should be used. By default, *lenient matching* is used, which means excess whitespace (*ignorable whitespace*) will not be taken into account for matching test results. Setting this flag to `true` will perform exact *character-by-character* matching. |
|`%EXCEPTION`                | Exception to be expected, if we expect template execution to raise an exception. Either this or the `%OUTPUT` directive must be specified.  |
|`%EXCEPTION_MESSAGE_PATTERN`| Pattern (in `java.util.regex.Pattern` syntax) expected to match the message of the exception raised during template execution. This directive needs the `%EXCEPTION` directive to be specified too. |

*Inheritance:*

| Name                       | Description |
|----------------------------|-------------|
|`%EXTENDS`                  | Test specification (in a format understandable by the implementation of `ITestableResolver` being used) from which this test must inherit all its directives, overriding only those that are explicitly specified in the current test along with this `%EXTENDS` directive.<br />Example: `%EXTENDS test/bases/base1.test` |



### Extending the standard test resolution mechanism ###

In fact, the way all these directives are evaluated can be changed and extended by specifying different *directive evaluators*

IStandardTestBuilder, IStandardTestEvaluator, IStandardTestFieldEvaluator, IStandardTestReader

