## Описание синтаксиса правил

Пользовательские DSL-правила на Python-подобном языке JSA DSL позволяют задавать:
* PVO — потенциально уязвимые операции;
* TDE — точки входа пользовательских данных;
* Filter — фильтрующие функции, которые обрабатывают входные пользовательские параметры: получая потенциально опасные значения, они возвращают их отфильтрованными и, соответственно, безопасными.

**Блок namespace**

Корневым элементом является блок `namespace`, который соответствует модулю в конкретном языке программирования (например, JavaScript-модулю, Java-пакету или пространству имен .NET):

```
namespace example:
   class_definitions
   function_definitions
```

В `namespace` объявляются классы и функции. Один и тот же `namespace` может быть объявлен в нескольких файлах, но все объявления классов и функций внутри этих файлов должны быть уникальными.

**Объявление класса**

При создании класса можно указать один или несколько базовых классов. Это позволяет наследовать их атрибуты и методы:

```
class ClassName[(Base[, Base]*)]:
   function_definitions
```

**Объявление функций**

При объявлении функции в динамически типизированных языках указание типов параметров не требуется, тогда как в статически типизированных оно обязательно:

```
def [returnType] funcName([[paramType] paramName[, [paramType] paramName]*]):
   function_body
```

Рекомендуется явно указывать тип возвращаемого значения даже в динамически типизированных языках. Это улучшает анализ кода, особенно в конструкциях с цепочкой вызовов вида `new DslClass().Bar().Baz()`.

Если функция ничего не возвращает (`void`, `undefined`, `None`), тип можно не указывать.

Ссылка на тип может быть точечным именем или строкой в обратных кавычках. Обратные кавычки нужны для имён, которые нельзя выразить обычным DSL-идентификатором:

```
def `java.util.Map<java.lang.String, ?>` values():
  return Taint
```

**Тело функции**

Тело функции может содержать один или несколько операторов `Detect`, `DetectReachability` и необязательный оператор `return`.

Оператор `Detect` используется для указания уязвимого параметра и описания потенциальной уязвимости:

```
Detect (param, vulnType, vulnGrammar)
```

где:
* `vulnType` — [тип возможной уязвимости](#Соответствие-уязвимостей-и-их-грамматического-контекста);
* `vulnGrammar` — [тип грамматического контекста](#Соответствие-уязвимостей-и-их-грамматического-контекста).

`DetectReachability` помечает точку кода для проверки её достижимости. Это оператор без аргументов и скобок:

```
def target():
  DetectReachability
```

Оператор `return` возвращает результат работы функции:
* один из параметров: `return foo`;
* отфильтрованный параметр: `return foo.filter(SqlCommon)`;
* параметр с несколькими фильтрами: `return foo.filter(SqlCommon, SQLInjection)`;
* цепочку фильтров: `return foo.filter(SqlCommon).filter(CrossSiteScripting)`;
* параметр, отмеченный как потенциально опасный (Taint): `return Taint`.

В `filter` можно передавать обычные типы уязвимостей, PVO-грамматики, типы слабостей (weakness) и секретов (secret), включённые во встроенные каталоги языкового сервера.
Отдельные DSL-вызовы `DetectWeakness` и `DetectSecret` пока не входят в поддерживаемый runtime-контракт.

## Примеры правил

Ниже приведены примеры DSL-правил для динамически и статически типизированных языков.

**Пример для динамически типизированных языков**

```
namespace dslClassTest: # объявление модуля
  class PvoClass: # объявление класса
    def pvoFunc(self, param): # объявление функции
      Detect(param, SQLInjection, SqlCommon) # детализация уязвимого параметра функции и параметров потенциальной уязвимости

  class PvoInheritor(PvoClass): # объявление класса-наследника, класс-родитель должен быть объявлен в этом же модуле
    pass # без дополнительных методов

namespace dslTest: # объявление другого модуля
  def pvoFunc(param): # объявление потенциально уязвимой функции модуля
      Detect(param, SQLInjection, SqlCommon) # детализация уязвимости
  def taintFunc(): # объявление функции TDE
    return Taint # функция возвращает потенциально опасное значение

namespace dslFilter: # объявление модуля с фильтрующими функциями
  def filterSqlCommon(foo): # добавление функции-фильтра
    return foo.filter(SqlCommon) # функция возвращает значение параметра foo, сделав его безопасным для инъекций в грамматику SqlCommon

  def filterSQLInjection(foo): # добавление функции-фильтра
    return foo.filter(SQLInjection) # функция возвращает значение параметра foo, сделав его безопасным для типа уязвимостей SQLInjection
```

**Пример для статически типизированных языков**

```
namespace java.smoke:
  class PvoClass:
    def pvoMethod(String param): # тип параметра, если ничего не возвращается, то он считается void или аналогом
      Detect(param, SQLInjection, SqlCommon)

  class TaintClass:
    def String getTaint(): # возвращаемый тип
      return Taint

  class Filter:
    def String filterSqlCommon(String foo):
      return foo.filter(SqlCommon)

    def String filterSQLInjection(String foo):
      return foo.filter(SQLInjection)
```

## Соответствие уязвимостей и их грамматического контекста

Каждому типу уязвимости, обнаруживаемой анализатором JSA, соответствует рекомендуемый грамматический контекст — окружение или условия, при которых уязвимость может существовать в системе. В таблицах ниже приведены соответствия, опубликованные для поддерживаемых анализаторов.

Языковой сервер отдельно проверяет, доступен ли идентификатор в конкретной конструкции DSL. В частности, weakness- и secret-типы можно использовать как аргументы `filter`, но runtime DSL пока не принимает их в качестве аргумента `vulnType` оператора `Detect`, поскольку для них не реализованы отдельные операции.

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
