# LGTM Enterprise 1.25 CodeQL release notes

Version 1.25 of LGTM Enterprise was released on September 9, 2020. Detailed release notes and download archives can be found on https://github.com/Semmle/lgtm-enterprise/releases. The following release notes describe changes to the CodeQL analysis, which is used by LGTM to generate alerts. 

### Table of contents
  - [Changes to CodeQL](#changes-to-codeql)
  - [Changes to Go analysis](#changes-to-go-analysis)
  - [Changes to C/C++ analysis](#changes-to-cc-analysis)
  - [Changes to C# analysis](#changes-to-c-analysis)
  - [Changes to Java analysis](#changes-to-java-analysis)
  - [Changes to Javascript analysis](#changes-to-javascript-analysis)
  - [Changes to Python analysis](#changes-to-python-analysis)

## Changes to CodeQL

* CodeQL now supports the definition of new types as type unions. This
    feature currently allows unions of branches from an already existing
    algebraic data type and unions of database types.

## Changes to Go analysis

* Basic support for the [Go-restful](https://github.com/emicklei/go-restful) HTTP library has been added, which
  may lead to more results from the security queries.
* Basic support for the [Gorm](https://github.com/go-gorm/gorm) ORM library has been added (specifically, its SQL statement building facilities), which
  may lead to more results from the security queries.
* Basic support for the [Sqlx](https://github.com/jmoiron/sqlx) database access library has been added, which
  may lead to more results from the security queries.
* Basic support for the [Json-iterator](https://github.com/json-iterator/go) JSON library has been added, which
  may lead to more results from the security queries.
* Query "Use of insecure HostKeyCallback implementation" (`go/insecure-hostkeycallback`) is promoted from experimental status. This checks for insecurely omitting SSH host-key verification.
* Query "Insecure TLS configuration" (`go/insecure-tls`) is promoted from experimental status. This checks for use of insecure SSL/TLS versions and cipher suites.
* New query "Missing error check" (`go/missing-error-check`) added. This checks for dangerous pointer dereferences when an accompanying error value returned from a call has not been checked.
* The query "Unreachable statement" (`go/unreachable-statement`) now tolerates more unreachable return statements, which can often be required in Go following a function call that cannot return. Newly tolerated statements include `return true`, `return MyStruct{0, true}`, and any return when the return value has type `error`. This eliminates some nuisance results.
* Taint tracking through `range` statements has been improved, which may cause more results from the security queries.
* The query "Open URL redirect" (`go/unvalidated-url-redirection`) now recognizes more problematic fields of `URL` objects, allowing it to flag more results.
* The query "Clear-text logging of sensitive information" has been improved to recognize more sources of sensitive data, which may lead to more alerts. The query is now also more precise, which may reduce the number of false positives.
* A bug has been fixed that could cause the analysis not to terminate in the presence of cycles through embedded struct fields.
* Resolution of method calls through interfaces has been improved, resulting in more precise call-graph information, which in turn may eliminate false positives from the security queries.
* The query "Reflected cross-site scripting" has been improved to more correctly determine whether
  an HTML mime type will be sniffed, which should lead to more accurate results.
* The query "Email injection" (`go/email-injection`) has been moved out of the experimental folder. The query detects when untrusted input can be incorporated directly into an email.
* The extractor now attempts to extract the AST of all dependencies that are related to the packages passed explicitly on the commandline, which is determined by using the module root or, if not using modules, the directory containing the source for those packages. In particular, this means if a package passed to the extractor depends on another package inside the same module, the dependency's AST will now be extracted.
* The query "Open URL redirect" (`go/unvalidated-url-redirection`) now recognizes values returned by method `http.Request.FormValue` as possibly user controlled, allowing it to flag more true positive results.
* Modeling of several WebSocket libraries has been added, which may lead to more results from the
  security queries.
* Modeling of the `go.mongodb.org/mongo-driver/mongo` package has been added, which may lead to more
  results from the security queries.
* A new query "Redundant call to recover" (`go/redundant-recover`) has been added. The query detects calls to `recover` that have no effect.
* Modeling of the standard `io` library has been improved, which may lead to more results from the
  security queries.
* The queries "Uncontrolled data used in path expression" and "Arbitrary file write during zip
  extraction ("zip slip")" have been improved to recognize more file APIs, which may lead to more
  alerts.
* The query "Reflected cross-site scripting" has been improved to recognize more cases where the
  value should be considered to be safe, which should lead to fewer false positive results.
* The data-flow library has been improved, which affects and improves most security queries. In particular,
  flow through functions involving nested field reads and writes is now modeled more fully.
* Basic support for the [Mux](https://github.com/gorilla/mux/) HTTP library has been added, which
  may lead to more results from the security queries.
* The query "Clear-text logging of sensitive information" has been improved to recognize more logging APIs, which may lead to more alerts.
* Basic support for the [Macaron](https://go-macaron.com/) HTTP library has been added, which may lead to more results from the security queries.
* The query "Bad redirect check" (`go/bad-redirect-check`) now requires that the checked variable is actually used in a redirect as opposed to relying on a name-based heuristic. This eliminates some false positive results, and adds more true positive results.


## Changes to C/C++ analysis

* The library `VCS.qll` and all queries that imported it have been removed.
* The data-flow library has been improved, which affects most security queries by potentially
  adding more results. Flow through functions now takes nested field reads/writes into account.
  For example, the library is able to track flow from `taint()` to `sink()` via the method
  `getf2f1()` in
  ```c
  struct C {
      int f1;
  };

  struct C2
  {
      C f2;

      int getf2f1() {
          return f2.f1; // Nested field read
      }

      void m() {
          f2.f1 = taint();
          sink(getf2f1()); // NEW: taint() reaches here
      }
  };
  ```
* The security pack taint tracking library (`semmle.code.cpp.security.TaintTracking`) now considers that equality checks may block the flow of taint.  This results in fewer false positive results from queries that use this library.
* The length of a tainted string (such as the return value of a call to `strlen` or `strftime` with tainted parameters) is no longer itself considered tainted by the `models` library.  This leads to fewer false positive results in queries that use any of our taint libraries.
* The Uncontrolled format string querys (`cpp/tainted-format-string`, `cpp/tainted-format-string-through-global`) are now displayed by default on LGTM
* Index initializers, of the form `{ [1] = "one" }`, are extracted correctly. Previously, the kind of the
  expression was incorrect, and the index was not extracted.

## Changes to C# analysis

* The class `UnboundGeneric` has been refined to only be those declarations that actually
  have type parameters. This means that non-generic nested types inside constructed types,
  such as `A<int>.B`, no longer are considered unbound generics. (Such nested types do,
  however, still have relevant `.getSourceDeclaration()`s, for example `A<>.B`.)
* The data-flow library has been improved, which affects most security queries by potentially
  adding more results:
  - Flow through methods now takes nested field reads/writes into account.
    For example, the library is able to track flow from `"taint"` to `Sink()` via the method
    `GetF2F1()` in
    ```csharp
    class C1
    {
        string F1;
    }

    class C2
    {
        C1 F2;

        string GetF2F1() => F2.F1; // Nested field read

        void M()
        {
            F2 = new C1() { F1 = "taint" };
            Sink(GetF2F1()); // NEW: "taint" reaches here
        }
    }
    ```
  - Flow through collections is now modeled precisely. For example, instead of modeling an array
    store `a[i] = x` as a taint-step from `x` to `a`, we now model it as a data-flow step that
    stores `x` into `a`. To get the value back out, a matching read step must be taken.

    For source-code based data-flow analysis, the following constructs are modeled as stores into
    collections:
    - Direct array assignments, `a[i] = x`.
    - Array initializers, `new [] { x }`.
    - C# 6-style array initializers, `new C() { Array = { [i] = x } }`.
    - Call arguments that match a `params` parameter, where the C# compiler creates an array under-the-hood.
    - `yield return` statements.

    The following source-code constructs read from a collection:
    - Direct array reads, `a[i]`.
    - `foreach` statements.

    For calls out to library code, existing flow summaries have been refined to precisely
    capture how they interact with collection contents. For example, a call to
    `System.Collections.Generic.List<T>.Add(T)` stores the value of the argument into the
    qualifier, and a call to `System.Collections.Generic.List<T>.get_Item(int)` (that is, an
    indexer call) reads contents out of the qualifier. Moreover, the effect of
    collection-clearing methods such as `System.Collections.Generic.List<T>.Clear()` is now
    also modeled.

# Changes to Java analysis

* The data-flow library has been improved, which affects most security queries by potentially
  adding more results. Flow through methods now takes nested field reads/writes into account.
  For example, the library is able to track flow from `"taint"` to `sink()` via the method
  `getF2F1()` in
  ```java
  class C1 {
    String f1;
    C1(String f1) { this.f1 = f1; }
  }

  class C2 {
    C1 f2;
    String getF2F1() {
        return this.f2.f1; // Nested field read
    }
    void m() {
        this.f2 = new C1("taint");
        sink(this.getF2F1()); // NEW: "taint" reaches here
    }
  }
  ```

# Changes to JavaScript analysis

* Support for the following frameworks and libraries has been improved:
  - [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
  - [bluebird](http://bluebirdjs.com/)
  - [express](https://www.npmjs.com/package/express)
  - [execa](https://www.npmjs.com/package/execa)
  - [fancy-log](https://www.npmjs.com/package/fancy-log)
  - [fastify](https://www.npmjs.com/package/fastify)
  - [foreground-child](https://www.npmjs.com/package/foreground-child)
  - [fstream](https://www.npmjs.com/package/fstream)
  - [jGrowl](https://github.com/stanlemon/jGrowl)
  - [jQuery](https://jquery.com/)
  - [marsdb](https://www.npmjs.com/package/marsdb)
  - [micro](https://www.npmjs.com/package/micro/)
  - [minimongo](https://www.npmjs.com/package/minimongo/)
  - [mssql](https://www.npmjs.com/package/mssql)
  - [mysql](https://www.npmjs.com/package/mysql)
  - [npmlog](https://www.npmjs.com/package/npmlog)
  - [opener](https://www.npmjs.com/package/opener)
  - [pg](https://www.npmjs.com/package/pg)
  - [sequelize](https://www.npmjs.com/package/sequelize)
  - [spanner](https://www.npmjs.com/package/spanner)
  - [sqlite](https://www.npmjs.com/package/sqlite)
  - [ssh2-streams](https://www.npmjs.com/package/ssh2-streams)
  - [ssh2](https://www.npmjs.com/package/ssh2)
  - [vue](https://www.npmjs.com/package/vue)
  - [yargs](https://www.npmjs.com/package/yargs)
  - [webpack-dev-server](https://www.npmjs.com/package/webpack-dev-server)
* TypeScript 3.9 is now supported.
* TypeScript code embedded in HTML and Vue files is now extracted and analyzed.
* The analysis of sanitizers has improved, leading to more accurate
  results from the security queries.
* A library `semmle.javascript.explore.CallGraph` has been added to help write queries for exploring the call graph.
* Added data flow for `Map` and `Set`, and added matching type-tracking steps that can accessed using the `CollectionsTypeTracking` module.
* The data-flow node representing a parameter or destructuring pattern is now always the `ValueNode` corresponding to that AST node. This has a few consequences:
  - `Parameter.flow()` now gets the correct data flow node for a parameter. Previously this had a result, but the node was disconnected from the data flow graph.
  - `ParameterNode.asExpr()` and `.getAstNode()` now gets the parameter's AST node, whereas previously it had no result.
  - `Expr.flow()` now has a more meaningful result for destructuring patterns. Previously this node was disconnected from the data flow graph. Now it represents the values being destructured by the pattern.
* The global data-flow and taint-tracking libraries now model indirect parameter accesses through the `arguments` object in some cases, which may lead to additional results from some of the security queries, particularly "Prototype pollution in utility function".
* The predicates `Type.getProperty()` and variants of `Type.getMethod()` have been deprecated due to lack of use-cases. Looking up a named property of a static type is no longer supported, favoring faster extraction times instead.

### Changes to queries


| **Query**                      | **Expected impact**          | **Change**                                                                |
|--------------------------------|------------------------------|---------------------------------------------------------------------------|
| Client-side cross-site scripting (`js/xss`) | Fewer results | This query now recognizes additional safe patterns of constructing HTML. |
| Client-side URL redirect (`js/client-side-unvalidated-url-redirection`) | Fewer results | This query now recognizes additional safe patterns of doing URL redirects. |
| Code injection (`js/code-injection`) | More results | More potential vulnerabilities involving NoSQL code operators are now recognized. |
| Exception text reinterpreted as HTML (`js/exception-xss`) | Rephrased and changed visibility | Rephrased name and alert message. Severity lowered from error to warning. Results are now shown on LGTM by default. |
| Expression has no effect (`js/useless-expression`) | Fewer results | This query no longer flags an expression when that expression is the only content of the containing file. |
| Hard-coded credentials (`js/hardcoded-credentials`) | More results | This query now recognizes hard-coded credentials sent via HTTP authorization headers. |
| Incomplete URL scheme check (`js/incomplete-url-scheme-check`) | More results | This query now recognizes additional url scheme checks. |
| Insecure randomness (`js/insecure-randomness`) | Fewer results | This query now recognizes when an insecure random value is used as a fallback when secure random values are unsupported. |
| Misspelled variable name (`js/misspelled-variable-name`) | Message changed | The message for this query now correctly identifies the misspelled variable in additional cases. |
| Non-linear pattern (`js/non-linear-pattern`) | Fewer duplicates and message changed | This query now generates fewer duplicate alerts and has a clearer explanation in case of type annotations used in a pattern. |
| Prototype pollution in utility function (`js/prototype-pollution-utility`) | More results | This query now recognizes additional utility functions as vulnerable to prototype polution. |
| Uncontrolled command line (`js/command-line-injection`) | More results | This query now recognizes additional command execution calls. |
| Uncontrolled data used in path expression (`js/path-injection`) | More results | This query now recognizes additional file system calls. |
| Uncontrolled data used in path expression (`js/path-injection`) | Fewer results | This query no longer flags paths that have been checked to be part of a collection. |
| Unknown directive (`js/unknown-directive`) | Fewer results | This query no longer flags directives generated by the Babel compiler. |
| Unneeded defensive code (`js/unneeded-defensive-code`) | Fewer false-positive results | This query now recognizes checks meant to handle the `document.all` object. |
| Unused property (`js/unused-property`) | Fewer results | This query no longer flags properties of objects that are operands of `yield` expressions. |
| Zip Slip (`js/zipslip`) | More results | This query now recognizes additional vulnerabilities. |

The following low-precision queries are no longer run by default on LGTM (their results already were not displayed):

  - `js/angular/dead-event-listener`
  - `js/angular/unused-dependency`
  - `js/bitwise-sign-check`
  - `js/comparison-of-identical-expressions`
  - `js/conflicting-html-attribute`
  - `js/ignored-setter-parameter`
  - `js/jsdoc/malformed-param-tag`
  - `js/jsdoc/missing-parameter`
  - `js/jsdoc/unknown-parameter`
  - `js/json-in-javascript-file`
  - `js/misspelled-identifier`
  - `js/nested-loops-with-same-variable`
  - `js/node/cyclic-import`
  - `js/node/unused-npm-dependency`
  - `js/omitted-array-element`
  - `js/return-outside-function`
  - `js/single-run-loop`
  - `js/too-many-parameters`
  - `js/unused-property`
  - `js/useless-assignment-to-global`

## Changes to Python analysis

* Importing `semmle.python.web.HttpRequest` will no longer import `UntrustedStringKind` transitively. `UntrustedStringKind` is the most commonly used non-abstract subclass of `ExternalStringKind`. If not imported (by one mean or another), taint-tracking queries that concern `ExternalStringKind` will not produce any results. Please ensure such queries contain an explicit import (`import semmle.python.security.strings.Untrusted`).
