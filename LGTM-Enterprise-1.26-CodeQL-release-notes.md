# LGTM Enterprise 1.26 CodeQL release notes

Version 1.26 of LGTM Enterprise was released on December 16th, 2020. Detailed release notes and download archives can be found on https://github.com/Semmle/lgtm-enterprise/releases. The following release notes describe changes to the CodeQL analysis, which is used by LGTM to generate alerts. 

The version of the CodeQL CLI corresponding to this release is v2.3.4, which can be downloaded from https://github.com/github/codeql-cli-binaries/releases.

### Table of contents
  - [Changes to Go analysis](#changes-to-go-analysis)
  - [Changes to C/C++ analysis](#changes-to-cc-analysis)
  - [Changes to C# analysis](#changes-to-c-analysis)
  - [Changes to Java analysis](#changes-to-java-analysis)
  - [Changes to Javascript analysis](#changes-to-javascript-analysis)
  - [Changes to Python analysis](#changes-to-python-analysis)

## Changes to Go analysis

* The query "Suspicious characters in a regular expression" has been improved to recognize raw string literals, which should lead to fewer false positives.
* Added basic support for the Revel web framework.
* Added support for the Spew deep pretty-printing framework. This may cause the `go/clear-text-logging` query to return more results when sensitive data is exposed using this library.
* The accuracy of the `go/allocation-size-overflow` query was improved, excluding more false-positives in which a small array could be mistaken for one of unbounded size.
* Added support for the `golang.org/x/net/context` package, which was already supported under its modern standard-library name `context`.
* A new query `go/stack-trace-exposure` has been added. The query flags exposure of a stack trace to a remote party.
* The extractor now only extracts go.mod files belonging to extracted packages. In particular, vendored go.mod files will no longer be extracted unless the vendored package is explicitly passed to the extractor. This will remove unexpected `GoModExpr` and similar expressions seen by queries.
* Add/improve taint-tracking models for 63 Go standard library packages. This means that all queries that track tainted data may produce more results; these include queries scanning for cross-site scripting vulnerabilities and SQL injection vulnerabilities among others.
* A new query `go/suspicious-character-in-regex` has been added. The query flags uses of `\b` and `\a` in regular expressions, where a character class was likely intended.
* Added support for the Echo web framework
* Added support for the Chi web framework
* Splitting a string by whitespace or a colon is now considered sanitizing by the `go/clear-text-logging` query, because this is frequently used to split a username and password or other secret.
* The query "Reflected cross-site scripting" (`go/reflected-xss`) now recognizes more cases of JSON marshaled data, which cannot serve as a vector for an XSS attack. This may reduce false-positive results for this query.
* Support for the [GORM](https://github.com/go-gorm/gorm) ORM library (specifically, its SQL
  statement building facilities) has been improved, which may lead to more results from the
  security queries.
* The query "Size computation for allocation may overflow" has been improved to recognize more
  cases where the value should be considered to be safe, which should lead to fewer false
  positive results.
* Taint is now propagated across protocol buffer ("protobuf")  marshalling and unmarshalling operations. This may result in more results from existing queries where the protocol buffer format is used.
* Basic support for the [Gin](https://github.com/gin-gonic/gin) HTTP library has been added (extending UntrustedFlowSource), which may lead to more results from the security queries.
* The query "Use of constant `state` value in OAuth 2.0 URL" (`go/constant-oauth2-state`) has been promoted from experimental status. This checks for use of a constant state value in generating an OAuth2 redirect URL, which may open the way for a CSRF attack.
* Query "Redundant check for negative value" (`go/negative-length-check`) has been expanded to consider unsigned integers, along
  with the return values of `len` and `cap` which it already handled. It has also been renamed to match its expanded role.
* Query "Incorrect integer conversion" (`go/incorrect-integer-conversion`) is promoted from experimental status. This checks for parsing a string to an integer and then assigning it to an integer type of a smaller bit size.
* The Go extractor now supports build tracing, allowing users to supply a build command when
  creating databases with the CodeQL CLI or via configuration. It currently only supports projects
  that use Go modules. To opt-in, set the environment variable `CODEQL_EXTRACTOR_GO_BUILD_TRACING`
  to `on`, or supply a build command.

## Changes to C/C++ analysis

* The 'Not enough memory allocated for pointer type' (`cpp/allocation-too-small`) and 'Not enough memory allocated for array of pointer type' (`cpp/suspicious-allocation-size`) queries have been improved. Previously some allocations would be reported by both queries, and this no longer occurs. More allocation functions are now understood by both queries.
* The `cpp/wrong-type-format-argument` and `cpp/non-portable-printf` queries have been hardened so that they do not produce nonsensical results on databases that contain errors (specifically the `ErroneousType`).
* The `SimpleRangeAnalysis` library has gained support for several language
  constructs it did not support previously. These improvements primarily affect
  the queries `cpp/constant-comparison`, `cpp/comparison-with-wider-type`, and
  `cpp/integer-multiplication-cast-to-long`. The newly supported language
  features are:
    * Multiplication of unsigned numbers.
    * Multiplication by a constant.
    * Reference-typed function parameters.
    * Comparing a variable not equal to an endpoint of its range, thus narrowing the range by one.
    * Using `if (x)` or `if (!x)` or similar to test for equality to zero.

### Changes to existing queries

| **Query**                  | **Expected impact**    | **Change**                                                       |
|----------------------------|------------------------|------------------------------------------------------------------|
| Declaration hides parameter (`cpp/declaration-hides-parameter`) | Fewer false positive results | False positives involving template functions have been fixed. |
| Inconsistent direction of for loop (`cpp/inconsistent-loop-direction`) | Fewer false positive results | The query now accounts for intentional wrapping of an unsigned loop counter. |
| Overflow in uncontrolled allocation size (`cpp/uncontrolled-allocation-size`) | | The precision of this query has been decreased from "high" to "medium". As a result, the query is still run but results are no longer displayed on LGTM by default. |
| Comparison result is always the same (`cpp/constant-comparison`) | More correct results | Bounds on expressions involving multiplication can now be determined in more cases. |

### Changes to libraries

* The QL class `Block`, denoting the `{ ... }` statement, is renamed to `BlockStmt`.
* The models library now models many taint flows through `std::array`, `std::vector`, `std::deque`, `std::list` and `std::forward_list`.
* The models library now models many more taint flows through `std::string`.
* The models library now models many taint flows through `std::istream` and `std::ostream`.
* The models library now models some taint flows through `std::shared_ptr`, `std::unique_ptr`, `std::make_shared` and `std::make_unique`.
* The models library now models many taint flows through `std::pair`, `std::map`, `std::unordered_map`, `std::set` and `std::unordered_set`.
* The models library now models `bcopy`.
* The `SimpleRangeAnalysis` library can now be extended with custom rules. See
  examples in
  `cpp/ql/src/experimental/semmle/code/cpp/rangeanalysis/extensions/`.

## Changes to C# analysis

* The `toString()` predicate is now less verbose for CIL entities. The old `toString()` behavior
  can be recovered by using `toStringExtra()` and `getQualifiedName()`, respectively.
* Assertion methods are now taking into account methods with parameters that are annotated
  with `System.Diagnostics.CodeAnalysis.DoesNotReturnIfAttribute`. This change is expected to
  lead to higher precision in any query that relies on control flow graphs.
* Attribute extraction has been extended to extract attributes not only from source
  code, but from referenced assemblies too. This change may lead to more results in
  queries that rely on attributes. Note that, as more attributes might be extracted,
  the DB size might increase.
* The required key size for the query "Weak encryption: Insufficient key size" has been increased from 1024 to 2048.
* The extractor is now assembly-insensitive by default. This means that two entities
  with the same fully-qualified name are now mapped to the same entity in the resulting
  database, regardless of whether they belong to different assemblies. Assembly
  sensitivity can be reenabled by passing `--assemblysensitivetrap` to the extractor.
* Inferring the lengths of implicitely sized arrays is fixed. Previously, multi
  dimensional arrays were always extracted with the same length for each dimension.
  With the fix, the array sizes `2` and `1` are extracted for `new int[,]{{1},{2}}`.
  Previously `2` and `2` were extracted.
* Partial method bodies are extracted. Previously, partial method bodies were skipped.
* The Abstract Syntax Tree of C# files can be viewed in Visual Studio Code.

## Changes to Java analysis

* A new query "Insecure Bean Validation" (`java/insecure-bean-validation`) has been added. This query 
  finds server-side template injections caused by untrusted data flowing from a bean 
  property into a custom error message for a constraint validator. This 
  vulnerability can lead to arbitrary code execution.
* Some methods of the [Guava](https://guava.dev/) framework have been added as flow steps (specifically those of the [Splitter](https://guava.dev/releases/30.0-jre/api/docs/com/google/common/base/Splitter.html), [Joiner](https://guava.dev/releases/30.0-jre/api/docs/com/google/common/base/Joiner.html), and [Strings](https://guava.dev/releases/30.0-jre/api/docs/com/google/common/base/Strings.html) classes), which may lead to more results from the security queries.
* Exported Android `Intent`s have been added as sources of tainted data for all
  security queries.
* The methods of the [Spring Web MultipartRequest](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/multipart/MultipartRequest.html)
  class have been added as sources of remote user input, which may lead to more results from the security queries.
* Support for the [Hibernate ORM](https://hibernate.org/orm/) library (specifically, its Query
  creation methods) has been improved, which may lead to more results from the security queries.
* A new query "Detect JHipster Generator Vulnerability CVE-2019-16303"
  (`java/jhipster-prng`) has been added. This query finds weak random number generators
  in security-sensitive methods generated by a vulnerable version of JHipster.
* The query "Uncontrolled command line" (`java/command-line-injection`) has
  been improved to better distinguish between command injection and safe
  command arguments.
* The QL class `Block`, denoting the `{ ... }` statement, is renamed to `BlockStmt`.
* Several security queries have been refactored to make them easier to extend with additional
  sinks and/or taint steps. Sink definitions have generally been moved to importable libraries,
  which can then be extended in `Customizations.qll`.
* Data flow is now supported through Java 14 records.
* The string format queries now recognize the Java 14 `String.formatted` method.
* Virtual dispatch in data flow has been improved to take call-context-specific type
  improvements to instance arguments into account. This improves precision for certain
  code patterns involving heavy virtual dispatch.
* The query "Cross-site scripting" (`java/xss`) has been improved to recognize
  `PrintWriter.format` as an XSS sink.
* The query "Information exposure through a stack trace" (`java/stack-trace-exposure`) has been
  improved to report fewer false positives when `super.printStackTrace()` is called
  in an overridden method.
* Two new queries, "Untrusted data passed to external API" (`java/untrusted-data-to-external-api`)
  and "Frequency counts for external APIs that are used with untrusted data"
  (`java/count-untrusted-data-external-api`), have been added. These queries
  should not be run by default as they are designed to have a low "true
  positive" rate. However, they allow you to review the use of untrusted data
  in an application to find new security vulnerabilities that are not found by
  the default security queries, as well as identifying opportunities to improve
  or add modeling of taint steps and sinks.
* The query "Uncontrolled data used in path expression" (`java/path-injection`) has been
  improved to recognize more path creation entry points.
* The SQL injection queries have been improved to recognize unsafe jOOQ methods.
* Reads from `java.net.http.WebSocket` have been added as sources of tainted data for all
  security queries.
* The SQL injection queries have been improved to recognize MongoDB injection sinks.

## Changes to JavaScript analysis

### General improvements

* Angular-specific taint sources and sinks are now recognized by the security queries.

* Support for React has improved, with better handling of react hooks, react-router path parameters, lazy-loaded components, and components transformed using `react-redux` and/or `styled-components`.

* Dynamic imports are now analyzed more precisely.

* Support for the following frameworks and libraries has been improved:
  - [@angular/*](https://www.npmjs.com/package/@angular/core)
  - [AWS Serverless](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-function.html)
  - [Alibaba Serverless](https://www.alibabacloud.com/help/doc-detail/156876.htm)
  - [debounce](https://www.npmjs.com/package/debounce)
  - [bluebird](https://www.npmjs.com/package/bluebird)
  - [call-limit](https://www.npmjs.com/package/call-limit)
  - [classnames](https://www.npmjs.com/package/classnames)
  - [clsx](https://www.npmjs.com/package/clsx)
  - [express](https://www.npmjs.com/package/express)
  - [fast-json-stable-stringify](https://www.npmjs.com/package/fast-json-stable-stringify)
  - [fast-safe-stringify](https://www.npmjs.com/package/fast-safe-stringify)
  - [http](https://nodejs.org/api/http.html)
  - [javascript-stringify](https://www.npmjs.com/package/javascript-stringify)
  - [js-stringify](https://www.npmjs.com/package/js-stringify)
  - [json-stable-stringify](https://www.npmjs.com/package/json-stable-stringify)
  - [json-stringify-safe](https://www.npmjs.com/package/json-stringify-safe)
  - [json3](https://www.npmjs.com/package/json3)
  - [jQuery throttle / debounce](https://github.com/cowboy/jquery-throttle-debounce)
  - [lodash](https://www.npmjs.com/package/lodash)
  - [lodash.debounce](https://www.npmjs.com/package/lodash.debounce)
  - [lodash.throttle](https://www.npmjs.com/package/lodash.throttle)
  - [needle](https://www.npmjs.com/package/needle)
  - [object-inspect](https://www.npmjs.com/package/object-inspect)
  - [pretty-format](https://www.npmjs.com/package/pretty-format)
  - [react](https://www.npmjs.com/package/react)
  - [react-router-dom](https://www.npmjs.com/package/react-router-dom)
  - [react-redux](https://www.npmjs.com/package/react-redux)
  - [redis](https://www.npmjs.com/package/redis)
  - [redux](https://www.npmjs.com/package/redux)
  - [stringify-object](https://www.npmjs.com/package/stringify-object)
  - [styled-components](https://www.npmjs.com/package/styled-components)
  - [throttle-debounce](https://www.npmjs.com/package/throttle-debounce)
  - [underscore](https://www.npmjs.com/package/underscore)

* Analyzing files with the ".cjs" extension is now supported.
* ES2021 features are now supported.

### Changes to existing queries

| **Query**                      | **Expected impact**          | **Change**                                                                |
|--------------------------------|------------------------------|---------------------------------------------------------------------------|
| Potentially unsafe external link (`js/unsafe-external-link`) | Fewer results | This query no longer flags URLs constructed using a template system where only the hash or query part of the URL is dynamic. |
| Incomplete URL substring sanitization (`js/incomplete-url-substring-sanitization`) | More results | This query now recognizes additional URLs when the substring check is an inclusion check. |
| Ambiguous HTML id attribute (`js/duplicate-html-id`) | Results no longer shown | Precision tag reduced to "low". The query is no longer run by default. |
| Unused loop iteration variable (`js/unused-loop-variable`) | Fewer results | This query no longer flags variables in a destructuring array assignment that are not the last variable in the destructed array. |
| Unsafe shell command constructed from library input (`js/shell-command-constructed-from-input`) | More results | This query now recognizes more commands where colon, dash, and underscore are used. |
| Unsafe jQuery plugin (`js/unsafe-jquery-plugin`) | More results | This query now detects more unsafe uses of nested option properties. |
| Client-side URL redirect (`js/client-side-unvalidated-url-redirection`) | More results | This query now recognizes some unsafe uses of `importScripts()` inside WebWorkers. |
| Missing CSRF middleware (`js/missing-token-validation`) | More results | This query now recognizes writes to cookie and session variables as potentially vulnerable to CSRF attacks. |
| Missing CSRF middleware (`js/missing-token-validation`) | Fewer results | This query now recognizes more ways of protecting against CSRF attacks. |
| Client-side cross-site scripting (`js/xss`) | More results | This query now tracks data flow from `location.hash` more precisely. |


### Changes to libraries
* The predicate `TypeAnnotation.hasQualifiedName` now works in more cases when the imported library was not present during extraction.
* The class `DomBasedXss::Configuration` has been deprecated, as it has been split into `DomBasedXss::HtmlInjectionConfiguration` and `DomBasedXss::JQueryHtmlOrSelectorInjectionConfiguration`. Unless specifically working with jQuery sinks, subclasses should instead be based on `HtmlInjectionConfiguration`. To use both configurations in a query, see [Xss.ql](https://github.com/github/codeql/blob/main/javascript/ql/src/Security/CWE-079/Xss.ql) for an example.


## Changes to Python analysis

### Changes to existing queries

| **Query**                  | **Expected impact**    | **Change**                                                       |
|----------------------------|------------------------|------------------------------------------------------------------|
|`py/unsafe-deserialization` | Different results. | The underlying data flow library has been changed. See below for more details. |
|`py/path-injection` | Different results. | The underlying data flow library has been changed. See below for more details. |
|`py/command-line-injection` | Different results. | The underlying data flow library has been changed. See below for more details. |
|`py/reflective-xss` | Different results. | The underlying data flow library has been changed. See below for more details. |
|`py/sql-injection` | Different results. | The underlying data flow library has been changed. See below for more details. |
|`py/code-injection` | Different results. | The underlying data flow library has been changed. See below for more details. |

### Changes to libraries
* Some of the security queries now use the shared data flow library for data flow and taint tracking. This has resulted in an overall more robust and accurate analysis. The libraries mentioned below have been modelled in this new framework. Other libraries (e.g. the web framework `CherryPy`) have not been modelled yet, and this may lead to a temporary loss of results for these frameworks.
* Improved modelling of the following serialization libraries:
  - `PyYAML`
  - `dill`
  - `pickle`
  - `marshal`
* Improved modelling of the following web frameworks:
  - `Django` (Note that modelling of class-based response handlers is currently incomplete.)
  - `Flask`
* Support for Werkzeug `MultiDict`.
* Support for the [Python Database API Specification v2.0 (PEP-249)](https://www.python.org/dev/peps/pep-0249/), including the following libraries:
  - `MySQLdb`
  - `mysql-connector-python`
  - `django.db`
* Improved modelling of the following command execution libraries:
  - `Fabric`
  - `Invoke`
* Improved modelling of security-related standard library modules, such as `os`, `popen2`, `platform`, and `base64`.
* The original versions of the updated queries have been preserved [here](https://github.com/github/codeql/tree/main/python/ql/src/experimental/Security-old-dataflow).
* Added taint tracking support for string formatting through f-strings.
