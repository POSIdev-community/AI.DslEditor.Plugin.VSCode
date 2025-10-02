
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

**Тело функции**

Тело функции может содержать один или несколько операторов `Detect` и необязательный оператор `Return`.

Оператор `Detect` используется для указания уязвимого параметра и описания потенциальной уязвимости:

```
Detect (param, vulnType, vulnGrammar)
```

где:
* `vulnType` — тип возможной уязвимости;
* `vulnGrammar` — тип грамматического контекста.

Оператор `Return` возвращает результат работы функции:
* один из параметров: `return foo`;
* отфильтрованный параметр: `return foo.filter(SqlCommon)`;
* параметр, отмеченный как потенциально опасный (Taint): `return Taint`.

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

Согласно синтаксису DSL-правил, каждому типу обнаруживаемой уязвимости должен соответствовать контекст — окружение или условия, при которых уязвимость существует в системе.

На текущий момент для всех языков, поддерживающих создание пользовательских DSL-правил, действуют ограничения на соответствие типов уязвимостей и контекстов. Наборы допустимых комбинаций параметров `vulnType` и `vulnGrammar` приведны таблицах ниже.

**PHP**

<table><caption>Возможные комбинации параметров vulnType и vulnGrammar</caption><colgroup><col style="width: 65%;"/><col style="width: 35%;"/></colgroup><thead><tr><th align="left">

vulnType
</th><th align="left">

vulnGrammar
</th></tr></thead><tbody><tr><td align="left">

CrossSiteScripting
</td><td align="left">

HtmlText
</td></tr><tr><td align="left">

RemoteCodeExecution
</td><td align="left">

PhpCode


PhpInfo


PhpInfoBrackets


PhpAssert
</td></tr><tr><td align="left">

OSCommanding
</td><td align="left">

CommandLineArguments


CommandLineArgumentsSilent
</td></tr><tr><td align="left">

SQLInjection
</td><td align="left">

SqlCommon


Postgres


Sqlite


Sqlite3


IBase


Informix


Ingres


MsSql


OracleSql


Sybase
</td></tr><tr><td align="left">

ArbitraryFileCreation
</td><td align="left">

FileName


ArbitraryFileNameShort
</td></tr><tr><td align="left">

ArbitraryFileModification
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryFileReading
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryFileDeletion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryFileCopying
</td><td align="left">

FileName


ArbitraryFileCopyingDestination
</td></tr><tr><td align="left">

ArbitraryDirectoryListing
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryCreation
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryMoving
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryDeletion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

UnrestrictedFileUpload
</td><td align="left">

FileName
</td></tr><tr><td align="left">

LocalFileInclusion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

RemoteFileInclusion
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

ServerSideRequestForgery
</td><td align="left">

Host


SSRFHostWithPort


HttpUri


IBase


Postgres


Cubrid


DB2


Sqlsrv
</td></tr><tr><td align="left">

DeserializationOfUntrustedData     
</td><td align="left">

SerializedObject


SerializedArrayObject


PharArchivePath
</td></tr><tr><td align="left">

LdapInjection
</td><td align="left">

Ldap
</td></tr><tr><td align="left">

HttpResponseSplitting
</td><td align="left">

HttpHeaderValue
</td></tr><tr><td align="left">

OpenRedirect
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

LogForging
</td><td align="left">

LogEntry
</td></tr><tr><td align="left">

XmlExternalEntity
</td><td align="left">

XmlDocument


HttpUri
</td></tr><tr><td align="left">

SmtpHeaderInjection
</td><td align="left">

SmtpHeader
</td></tr><tr><td align="left">

XPathInjection
</td><td align="left">

XPath
</td></tr><tr><td align="left">

ResourceExhaustion
</td><td align="left">

ArbitraryIntData
</td></tr></tbody></table>


**Python**

<table><caption>Возможные комбинации параметров vulnType и vulnGrammar</caption><colgroup><col style="width: 65%;"/><col style="width: 35%;"/></colgroup><thead><tr><th align="left">

vulnType
</th><th align="left">

vulnGrammar
</th></tr></thead><tbody><tr><td align="left">

ArbitraryFileCreation
</td><td align="left">

FileName


ArbitraryIntData
</td></tr><tr><td align="left">

ArbitraryFileDeletion
</td><td align="left">

FileName


ArbitraryIntData
</td></tr><tr><td align="left">

ArbitraryFileModification
</td><td align="left">

FileName


ArbitraryIntData
</td></tr><tr><td align="left">

ArbitraryFileReading
</td><td align="left">

FileName


ArbitraryIntData
</td></tr><tr><td align="left">

ArbitraryDirectoryListing
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryCreation
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryMoving
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryDeletion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

CrossSiteScripting
</td><td align="left">

HtmlText
</td></tr><tr><td align="left">

SQLInjection
</td><td align="left">

SqlCommon
</td></tr><tr><td align="left">

OpenRedirect
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

OSCommanding
</td><td align="left">

CommandLineArguments
</td></tr><tr><td align="left">

RemoteCodeExecution
</td><td align="left">

PythonCode
</td></tr><tr><td align="left">

UncontrolledDataManipulation
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

LocalFileInclusion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

CookieInjection
</td><td align="left">

HttpCookieName


HttpCookieValue
</td></tr><tr><td align="left">

HttpHeaderInjection
</td><td align="left">

HttpHeaderName


HttpHeaderValue
</td></tr><tr><td align="left">

DeserializationOfUntrustedData     
</td><td align="left">

ArbitraryBinaryData
</td></tr><tr><td align="left">

LogForging
</td><td align="left">

LogEntry
</td></tr><tr><td align="left">

ServerSideRequestForgery
</td><td align="left">

HttpUri
</td></tr></tbody></table>


**Go**

<table><caption>Возможные комбинации параметров vulnType и vulnGrammar</caption><colgroup><col style="width: 65%;"/><col style="width: 35%;"/></colgroup><thead><tr><th align="left">

vulnType
</th><th align="left">

vulnGrammar
</th></tr></thead><tbody><tr><td align="left">

CrossSiteScripting
</td><td align="left">

HtmlText
</td></tr><tr><td align="left">

OpenRedirect
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

ServerSideRequestForgery
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

OSCommanding
</td><td align="left">

CommandLineArguments
</td></tr><tr><td align="left">

ArbitraryFileCreation
</td><td align="left">

FileName


ArbitraryStringData
</td></tr><tr><td align="left">

ArbitraryFileModification
</td><td align="left">

FileName


ArbitraryStringData
</td></tr><tr><td align="left">

ArbitraryFileReading
</td><td align="left">

FileName


ArbitraryStringData
</td></tr><tr><td align="left">

ArbitraryFileDeletion
</td><td align="left">

FileName


ArbitraryStringData
</td></tr><tr><td align="left">

ArbitraryDirectoryListing
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryCreation
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryMoving
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryDeletion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

PotentialPathTraversal
</td><td align="left">

FileName
</td></tr><tr><td align="left">

RemoteCodeExecution
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

SQLInjection
</td><td align="left">

SqlCommon
</td></tr><tr><td align="left">

InsecureDirectObjectReferences
</td><td align="left">

ArbitraryStringData


ArbitraryIntData
</td></tr><tr><td align="left">

LogForging
</td><td align="left">

LogEntry
</td></tr><tr><td align="left">

ResourceExhaustion
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

UncontrolledDataManipulation
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

IncorrectPermissionAssignmentForCriticalResource
</td><td align="left">

FileName
</td></tr><tr><td align="left">

CookieInjection
</td><td align="left">

HttpCookieName


HttpCookieValue
</td></tr><tr><td align="left">

InsecureCookieScope
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

PersistentCookieExposing
</td><td align="left">

ArbitraryDateString


ArbitraryIntData
</td></tr><tr><td align="left">

SensitiveCookieInHttpsSessionWithoutHttponlyAttribute
</td><td align="left">

HttpCookieValue
</td></tr><tr><td align="left">

SensitiveCookieInHttpsSessionWithoutSecureAttribute
</td><td align="left">

HttpCookieValue
</td></tr><tr><td align="left">

HttpHeaderInjection
</td><td align="left">

ArbitraryStringData


HttpHeaderName


HttpHeaderValue
</td></tr><tr><td align="left">

UnrestrictedFileUpload
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ExternalControlOfConfiguration
</td><td align="left">

ArbitraryStringData


FileName
</td></tr><tr><td align="left">

BufferOverflow
</td><td align="left">

ArbitraryIntData
</td></tr><tr><td align="left">

ImproperValidationOfArrayIndex
</td><td align="left">

ArbitraryIntData
</td></tr></tbody></table>


**Java**

<table><caption>Возможные комбинации параметров vulnType и vulnGrammar</caption><colgroup><col style="width: 65%;"/><col style="width: 35%;"/></colgroup><thead><tr><th align="left">

vulnType
</th><th align="left">

vulnGrammar
</th></tr></thead><tbody><tr><td align="left">

RemoteCodeExecution
</td><td align="left">

ArbitraryStringData


FileName


XmlDocument


JavaScriptCode


LogEntry
</td></tr><tr><td align="left">

OpenRedirect
</td><td align="left">

HttpUri


ArbitraryStringData
</td></tr><tr><td align="left">

XmlExternalEntity
</td><td align="left">

FileName


ArbitraryStringData


XmlDocument


HttpUri
</td></tr><tr><td align="left">

ServerSideRequestForgery
</td><td align="left">

IpAddress


HttpUri


FileName


Host


Port
</td></tr><tr><td align="left">

CrossSiteScripting
</td><td align="left">

HtmlText


ArbitraryStringData


JavaScriptCode


HtmlAttributeValue2


HttpUri
</td></tr><tr><td align="left">

OSCommanding
</td><td align="left">

CommandLineArguments


FileName
</td></tr><tr><td align="left">

SQLInjection
</td><td align="left">

SqlCommon
</td></tr><tr><td align="left">

JvmTrustBoundaryViolation
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

PotentialPathTraversal
</td><td align="left">

FileName


HttpUri
</td></tr><tr><td align="left">

PersistentCookieExposing
</td><td align="left">

ArbitraryIntData
</td></tr><tr><td align="left">

LocalFileInclusion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

UnrestrictedFileUpload
</td><td align="left">

FileName
</td></tr><tr><td align="left">

DeserializationOfUntrustedData
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

ArbitraryFileCreation
</td><td align="left">

ArbitraryStringData


FileName
</td></tr><tr><td align="left">

ArbitraryFileModification
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryFileReading
</td><td align="left">

ArbitraryStringData


FileName


HttpUri
</td></tr><tr><td align="left">

ArbitraryFileDeletion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

HttpHeaderInjection
</td><td align="left">

HttpHeaderName


HttpHeaderValue
</td></tr><tr><td align="left">

HttpRequestSplitting
</td><td align="left">

HttpHeaderValue
</td></tr><tr><td align="left">

HttpResponseSplitting
</td><td align="left">

HttpHeaderValue
</td></tr><tr><td align="left">

XPathInjection
</td><td align="left">

XPath
</td></tr><tr><td align="left">

LogForging
</td><td align="left">

LogEntry
</td></tr><tr><td align="left">

LdapInjection
</td><td align="left">

Ldap
</td></tr><tr><td align="left">

XQueryInjection
</td><td align="left">

Xquery
</td></tr><tr><td align="left">

ObjectInjection
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

InsecureDirectObjectReferences
</td><td align="left">

ArbitraryStringData


ArbitraryIntData
</td></tr><tr><td align="left">

InsufficientSessionExpiration
</td><td align="left">

ArbitraryIntData
</td></tr><tr><td align="left">

RemoteFileInclusion
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

JvmOrmInjection
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

CookieInjection
</td><td align="left">

HttpCookieName


HttpCookieValue
</td></tr><tr><td align="left">

CrossSiteRequestForgery
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

IncorrectPermissionAssignmentForCriticalResource
</td><td align="left">

FileName


FilePermissions
</td></tr><tr><td align="left">

ExternalControlOfConfiguration
</td><td align="left">

ArbitraryStringData
</td></tr></tbody></table>


**JavaScript/TypeScript**

<table><caption>Возможные комбинации параметров vulnType и vulnGrammar</caption><colgroup><col style="width: 65%;"/><col style="width: 35%;"/></colgroup><thead><tr><th align="left">

vulnType
</th><th align="left">

vulnGrammar
</th></tr></thead><tbody><tr><td align="left">

ArbitraryFileCreation
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryFileDeletion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryFileModification
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryFileCopying
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryFileReading
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryCreation
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryListing
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryMoving
</td><td align="left">

FileName
</td></tr><tr><td align="left">

ArbitraryDirectoryDeletion
</td><td align="left">

FileName
</td></tr><tr><td align="left">

CrossSiteRequestForgery
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

CrossSiteScripting
</td><td align="left">

HtmlText


HtmlAttributeValue2
</td></tr><tr><td align="left">

OSCommanding
</td><td align="left">

CommandLineArguments
</td></tr><tr><td align="left">

LogForging
</td><td align="left">

LogEntry
</td></tr><tr><td align="left">

OpenRedirect
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

ServerSideRequestForgery
</td><td align="left">

HttpUri
</td></tr><tr><td align="left">

SQLInjection
</td><td align="left">

SqlCommon


MySql
</td></tr><tr><td align="left">

RemoteCodeExecution
</td><td align="left">

JavaScriptCode


FileName
</td></tr><tr><td align="left">

NoSQLInjection
</td><td align="left">

MongoDB
</td></tr><tr><td align="left">

DeserializationOfUntrustedData     
</td><td align="left">

JsonNodeSerialize
</td></tr><tr><td align="left">

UncontrolledDataManipulation
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

ResourceExhaustion
</td><td align="left">

ArbitraryStringData
</td></tr><tr><td align="left">

CookieInjection
</td><td align="left">

HttpCookieName


HttpCookieValue
</td></tr><tr><td align="left">

HttpHeaderInjection
</td><td align="left">

HttpHeaderName


HttpHeaderValue
</td></tr><tr><td align="left">

IncorrectPermissionAssignmentForCriticalResource
</td><td align="left">

FileName


FilePermissions
</td></tr></tbody></table>

