## Rule syntax description

Custom DSL rules in the Python-like JSA DSL allow you to set:
* PVOs: potentially vulnerable operations.
* TDEs: taint data entry points.
* Filters: filtering functions that process user input parameters. If they receive potentially malicious values, they filter these out and return safe values.

**Namespace**

The root element is the `namespace` block, which matches a module in a particular programming language (for example, a JavaScript module, Java package, or .NET namespace):

```
namespace example:
   class_definitions
   function_definitions
```

Classes and functions are declared in the `namespace`. The same `namespace` can be declared in multiple files, but all declarations of classes and functions inside these files must be unique.

**Class declaration**

When creating a class, you can specify one or more base classes. This allows their attributes and methods to be inherited:

```
class ClassName[(Base[, Base]*)]:
   function_definitions
```

**Function declaration**

If you declare a function in a dynamically typed language, you do not need to specify parameter types (unlike in statically typed languages):

```
def [returnType] funcName([[paramType] paramName[, [paramType] paramName]*]):
   function_body
```

It is recommended that you explicitly specify the value type to be returned even in dynamically typed languages. This improves code analysis, especially for constructs with call chains like `new DslClass().Bar().Baz()`.

If the function does not return any value (`void`, `undefined`, `None`), the type can be omitted.

A type reference can be a dotted name or a value enclosed in backticks. Use the backtick form for names that cannot be represented as regular DSL identifiers:

```
def `java.util.Map<java.lang.String, ?>` values():
  return Taint
```

**Function body**

The function body can contain one or more `Detect` and `DetectReachability` statements and an optional `return` statement.

A `Detect` operator is used to specify a vulnerable parameter and describe a potential vulnerability:

```
Detect (param, vulnType, vulnGrammar)
```

where:
* `vulnType` is a [possible vulnerability type](#Matching-of-vulnerabilities-and-grammatical-contexts).
* `vulnGrammar` is the [grammatical context type](#Matching-of-vulnerabilities-and-grammatical-contexts).

`DetectReachability` marks a code location for reachability analysis. It is a statement without arguments or parentheses:

```
def target():
  DetectReachability
```

The `return` statement returns the function result:
* one parameter: `return foo`;
* a filtered parameter: `return foo.filter(SqlCommon)`;
* a parameter with multiple filters: `return foo.filter(SqlCommon, SQLInjection)`;
* a filter chain: `return foo.filter(SqlCommon).filter(CrossSiteScripting)`;
* a parameter marked as potentially malicious (`Taint`): `return Taint`.

`filter` accepts regular vulnerability types, PVO grammars, weakness types, and secret types known to the bundled JSA catalog. Separate `DetectWeakness` and `DetectSecret` statements are not part of the supported DSL runtime contract.

## Rule examples

Below, you can find examples of DSL rules for dynamically and statically typed languages.

**Example for dynamically typed languages**

```
namespace dslClassTest: # module declaration
  class PvoClass: # class declaration
    def pvoFunc(self, param): # function declaration
      Detect(param, SQLInjection, SqlCommon) # details of a vulnerable parameter and parameters of the potential vulnerability

  class PvoInheritor(PvoClass): # child class declaration; the parent class must be declared in the same module
    pass # without additional methods

namespace dslTest: # declaration of another module
  def pvoFunc(param): # declaration of a potentially vulnerable module function
      Detect(param, SQLInjection, SqlCommon) # details of the vulnerability
  def taintFunc(): # TDE function declaration
    return Taint # the function returns a potentially dangerous value

namespace dslFilter: # declaration of a module with filtering functions
  def filterSqlCommon(foo): # adding a filter function
    return foo.filter(SqlCommon) # the function returns the foo value and makes it safe to be injected into the SqlCommon grammar

  def filterSQLInjection(foo): # adding a filter function
    return foo.filter(SQLInjection) # the function returns the foo value and makes it safe for the SQLInjection vulnerability type
```

**Example for statically typed languages**

```
namespace java.smoke:
  class PvoClass:
    def pvoMethod(String param): # parameter type; if no value is returned, it is considered void or similar
      Detect(param, SQLInjection, SqlCommon)

  class TaintClass:
    def String getTaint(): # return type
      return Taint

  class Filter:
    def String filterSqlCommon(String foo):
      return foo.filter(SqlCommon)

    def String filterSQLInjection(String foo):
      return foo.filter(SQLInjection)
```

## Matching vulnerabilities and their grammatical contexts

Each vulnerability detected by a JSA analyzer has a recommended grammatical context: the environment or conditions in which the vulnerability can occur. The tables below reproduce the mappings published for the supported analyzers.

The language server validates whether an identifier is available in a particular DSL construct separately. In particular, weakness and secret types can be used as `filter` arguments, but are not accepted as the `vulnType` argument of `Detect` until the DSL runtime provides dedicated operations for them.

### .NET

| `vulnType` | `vulnGrammar` |
| --- | --- |
| `ArbitraryFileCreation` | `FileName` |
| `ArbitraryFileDeletion` | `FileName` |
| `ArbitraryFileModification` | `FileName` |
| `ArbitraryFileReading` | `FileName` |
| `CrossSiteScripting` | `HtmlText`<br>`HtmlTitle`<br>`HtmlAttributeList`<br>`HtmlAttributeValue1`<br>`HtmlAttributeValue2`<br>`JavaScriptCode`<br>`SerializedJsonData` |
| `DllInjection` | `FileName` |
| `ExternalControlOfConfiguration` | `ConfigurationKey`<br>`ConfigurationValue` |
| `HttpResponseSplitting` | `HttpHeaderValue` |
| `LdapInjection` | `Ldap`<br>`Linq` |
| `LocalFileInclusion` | `FileName` |
| `LogForging` | `LogEntry` |
| `OpenRedirect` | `HttpUri` |
| `OSCommanding` | `FileName` |
| `ServerSideRequestForgery` | `ConnectionStringItem`<br>`ConnectionStringText`<br>`NotApplicable`<br>`WebSocketUri` |
| `SQLInjection` | `MySql`<br>`NpgSql`<br>`OracleSql`<br>`SqlCe`<br>`SqlCommon`<br>`Sqlite` |
| `UnrestrictedFileUpload` | `FileName` |
| `XmlExternalEntity` | `XmlDocument` |
| `XPathInjection` | `XPath` |
| `XQueryInjection` | `Xquery` |

### Go

| `vulnType` | `vulnGrammar` |
| --- | --- |
| `ArbitraryDirectoryCreation` | `FileName` |
| `ArbitraryDirectoryDeletion` | `FileName` |
| `ArbitraryDirectoryListing` | `FileName` |
| `ArbitraryDirectoryMoving` | `FileName` |
| `ArbitraryFileCreation` | `ArbitraryStringData`<br>`FileName` |
| `ArbitraryFileDeletion` | `ArbitraryStringData`<br>`FileName` |
| `ArbitraryFileModification` | `ArbitraryStringData`<br>`FileName` |
| `ArbitraryFileReading` | `ArbitraryStringData`<br>`FileName` |
| `BufferOverflow` | `ArbitraryIntData` |
| `CookieInjection` | `HttpCookieName`<br>`HttpCookieValue` |
| `CrossSiteScripting` | `HtmlText` |
| `ExternalControlOfConfiguration` | `ArbitraryStringData`<br>`FileName` |
| `HttpHeaderInjection` | `ArbitraryStringData`<br>`HttpHeaderName`<br>`HttpHeaderValue` |
| `ImproperValidationOfArrayIndex` | `ArbitraryIntData` |
| `IncorrectPermissionAssignmentForCriticalResource` | `FileName` |
| `InsecureCookieScope` | `HttpUri` |
| `InsecureDirectObjectReferences` | `ArbitraryIntData`<br>`ArbitraryStringData` |
| `LogForging` | `LogEntry` |
| `OpenRedirect` | `HttpUri` |
| `OSCommanding` | `CommandLineArguments` |
| `PersistentCookieExposing` | `ArbitraryDateString`<br>`ArbitraryIntData` |
| `PotentialPathTraversal` | `FileName` |
| `RemoteCodeExecution` | `ArbitraryStringData` |
| `ResourceExhaustion` | `ArbitraryStringData` |
| `SensitiveCookieInHttpsSessionWithoutHttponlyAttribute` | `HttpCookieValue` |
| `SensitiveCookieInHttpsSessionWithoutSecureAttribute` | `HttpCookieValue` |
| `ServerSideRequestForgery` | `HttpUri` |
| `SQLInjection` | `SqlCommon` |
| `UncontrolledDataManipulation` | `ArbitraryStringData` |
| `UnrestrictedFileUpload` | `FileName` |

### Java

| `vulnType` | `vulnGrammar` |
| --- | --- |
| `ArbitraryFileCreation` | `ArbitraryStringData`<br>`FileName` |
| `ArbitraryFileDeletion` | `FileName` |
| `ArbitraryFileModification` | `FileName` |
| `ArbitraryFileReading` | `ArbitraryStringData`<br>`FileName`<br>`HttpUri` |
| `CrossSiteRequestForgery` | `HttpUri` |
| `CrossSiteScripting` | `ArbitraryStringData`<br>`HtmlAttributeValue2`<br>`HtmlText`<br>`HttpUri`<br>`JavaScriptCode` |
| `CookieInjection` | `HttpCookieName`<br>`HttpCookieValue` |
| `DeserializationOfUntrustedData` | `ArbitraryStringData` |
| `ExternalControlOfConfiguration` | `ArbitraryStringData` |
| `HttpHeaderInjection` | `HttpHeaderName`<br>`HttpHeaderValue` |
| `HttpRequestSplitting` | `HttpHeaderValue` |
| `HttpResponseSplitting` | `HttpHeaderValue` |
| `IncorrectPermissionAssignmentForCriticalResource` | `FileName`<br>`FilePermissions` |
| `InsecureDirectObjectReferences` | `ArbitraryIntData`<br>`ArbitraryStringData` |
| `InsufficientSessionExpiration` | `ArbitraryIntData` |
| `LdapInjection` | `Ldap` |
| `LocalFileInclusion` | `FileName` |
| `LogForging` | `LogEntry` |
| `ObjectInjection` | `HttpUri` |
| `OpenRedirect` | `ArbitraryStringData`<br>`HttpUri` |
| `OrmInjection` | `ArbitraryStringData` |
| `OSCommanding` | `CommandLineArguments`<br>`FileName` |
| `PersistentCookieExposing` | `ArbitraryIntData` |
| `PotentialPathTraversal` | `FileName`<br>`HttpUri` |
| `RemoteCodeExecution` | `ArbitraryStringData`<br>`FileName`<br>`JavaScriptCode`<br>`LogEntry`<br>`XmlDocument` |
| `RemoteFileInclusion` | `HttpUri` |
| `ServerSideRequestForgery` | `FileName`<br>`Host`<br>`HttpUri`<br>`IpAddress`<br>`Port` |
| `SQLInjection` | `SqlCommon` |
| `TrustBoundaryViolation` | `ArbitraryStringData` |
| `UnrestrictedFileUpload` | `FileName` |
| `XmlExternalEntity` | `ArbitraryStringData`<br>`FileName`<br>`HttpUri`<br>`XmlDocument` |
| `XPathInjection` | `XPath` |
| `XQueryInjection` | `Xquery` |

### JavaScript/TypeScript

| `vulnType` | `vulnGrammar` |
| --- | --- |
| `ArbitraryDirectoryCreation` | `FileName` |
| `ArbitraryDirectoryListing` | `FileName` |
| `ArbitraryDirectoryMoving` | `FileName` |
| `ArbitraryFileCopying` | `FileName` |
| `ArbitraryFileCreation` | `FileName` |
| `ArbitraryFileDeletion` | `FileName` |
| `ArbitraryFileModification` | `FileName` |
| `ArbitraryFileReading` | `FileName` |
| `ArbitraryDirectoryDeletion` | `FileName` |
| `CookieInjection` | `HttpCookieName`<br>`HttpCookieValue` |
| `CrossSiteRequestForgery` | `HttpUri` |
| `CrossSiteScripting` | `HtmlAttributeValue2`<br>`HtmlText` |
| `DeserializationOfUntrustedData` | `JsonNodeSerialize` |
| `HttpHeaderInjection` | `HttpHeaderName`<br>`HttpHeaderValue` |
| `IncorrectPermissionAssignmentForCriticalResource` | `FileName`<br>`FilePermissions` |
| `LogForging` | `LogEntry` |
| `NoSQLInjection` | `MongoDB` |
| `OpenRedirect` | `HttpUri` |
| `OSCommanding` | `CommandLineArguments` |
| `RemoteCodeExecution` | `JavaScriptCode`<br>`FileName` |
| `ResourceExhaustion` | `ArbitraryStringData` |
| `ServerSideRequestForgery` | `HttpUri` |
| `SQLInjection` | `SqlCommon`<br>`MySql` |
| `UncontrolledDataManipulation` | `ArbitraryStringData` |

### PHP

| `vulnType` | `vulnGrammar` |
| --- | --- |
| `ArbitraryDirectoryCreation` | `FileName` |
| `ArbitraryDirectoryDeletion` | `FileName` |
| `ArbitraryDirectoryListing` | `FileName` |
| `ArbitraryDirectoryMoving` | `FileName` |
| `ArbitraryFileCopying` | `ArbitraryFileCopyingDestination`<br>`FileName` |
| `ArbitraryFileCreation` | `ArbitraryFileNameShort`<br>`FileName` |
| `ArbitraryFileDeletion` | `FileName` |
| `ArbitraryFileModification` | `FileName` |
| `ArbitraryFileReading` | `FileName` |
| `CrossSiteScripting` | `HtmlText` |
| `DeserializationOfUntrustedData` | `PharArchivePath`<br>`SerializedObject`<br>`SerializedArrayObject` |
| `HttpResponseSplitting` | `HttpHeaderValue` |
| `LdapInjection` | `Ldap` |
| `LocalFileInclusion` | `FileName` |
| `LogForging` | `LogEntry` |
| `OpenRedirect` | `HttpUri` |
| `OSCommanding` | `CommandLineArguments`<br>`CommandLineArgumentsSilent` |
| `RemoteCodeExecution` | `PhpAssert`<br>`PhpCode`<br>`PhpInfo`<br>`PhpInfoBrackets` |
| `ResourceExhaustion` | `ArbitraryIntData` |
| `RemoteFileInclusion` | `HttpUri` |
| `ServerSideRequestForgery` | `Cubrid`<br>`DB2`<br>`Host`<br>`HttpUri`<br>`IBase`<br>`Postgres`<br>`Sqlsrv`<br>`SSRFHostWithPort` |
| `SmtpHeaderInjection` | `SmtpHeader` |
| `SQLInjection` | `IBase`<br>`Ingres`<br>`Informix`<br>`MsSql`<br>`OracleSql`<br>`Postgres`<br>`SqlCommon`<br>`Sqlite`<br>`Sqlite3`<br>`Sybase` |
| `UnrestrictedFileUpload` | `FileName` |
| `XmlExternalEntity` | `HttpUri`<br>`XmlDocument` |
| `XPathInjection` | `XPath` |

### Python

| `vulnType` | `vulnGrammar` |
| --- | --- |
| `ArbitraryDirectoryCreation` | `FileName` |
| `ArbitraryDirectoryDeletion` | `FileName` |
| `ArbitraryDirectoryListing` | `FileName` |
| `ArbitraryDirectoryMoving` | `FileName` |
| `ArbitraryFileCreation` | `ArbitraryIntData`<br>`FileName` |
| `ArbitraryFileDeletion` | `ArbitraryIntData`<br>`FileName` |
| `ArbitraryFileModification` | `ArbitraryIntData`<br>`FileName` |
| `ArbitraryFileReading` | `ArbitraryIntData`<br>`FileName` |
| `CookieInjection` | `HttpCookieName`<br>`HttpCookieValue` |
| `CrossSiteScripting` | `HtmlText` |
| `DeserializationOfUntrustedData` | `ArbitraryBinaryData` |
| `HttpHeaderInjection` | `HttpHeaderName`<br>`HttpHeaderValue` |
| `LocalFileInclusion` | `FileName` |
| `LogForging` | `LogEntry` |
| `ServerSideRequestForgery` | `HttpUri` |
| `SQLInjection` | `SqlCommon` |
| `OpenRedirect` | `HttpUri` |
| `OSCommanding` | `CommandLineArguments` |
| `RemoteCodeExecution` | `PythonCode` |
| `UncontrolledDataManipulation` | `ArbitraryStringData` |

### Ruby

| `vulnType` | `vulnGrammar` |
| --- | --- |
| `ArbitraryFileCreation` | `FileName` |
| `ArbitraryFileDeletion` | `FileName` |
| `ArbitraryFileModification` | `FileName` |
| `ArbitraryFileReading` | `FileName` |
| `CookieInjection` | `ArbitraryStringData`<br>`HttpCookieName`<br>`HttpCookieValue` |
| `CrossSiteScripting` | `HtmlText` |
| `DeserializationOfUntrustedData` | `SerializedJsonData` |
| `HttpHeaderInjection` | `ArbitraryStringData`<br>`HttpHeaderName`<br>`HttpHeaderValue` |
| `InsecureDirectObjectReferences` | `ArbitraryStringData` |
| `OpenRedirect` | `ArbitraryStringData`<br>`HttpHeaderName`<br>`HttpUri` |
| `OSCommanding` | `CommandLineArguments` |
| `SQLInjection` | `SqlCommon` |
| `UncontrolledDataManipulation` | `ArbitraryStringData` |
| `XmlExternalEntity` | `XmlDocument` |
| `XmlEntityExpansion` | `XmlDocument` |
