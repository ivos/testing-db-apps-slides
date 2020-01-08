## Testing DB apps <br> in Java / Clojure

<!-- .element: style="margin-bottom: 5rem;" -->

Practical guide to system tests

<!-- .element: style="margin-bottom: 5rem;" -->

> Ivo Maixner  
> ivo.maixner@gmail.com


^^^^

## Agenda

- System tests challenge
- Common anti-patterns
- How to
- Benefits


^^^^

<!-- .slide: data-transition="fade" -->

## DB app?

- <!-- .element: class="fragment" --> Backend with business logic
- <!-- .element: class="fragment" --> Data in DB
- <!-- .element: class="fragment" --> API: REST, msg
- <!-- .element: class="fragment" --> Optional UI


^^

<!-- .slide: data-transition="fade" -->

## System tests defined

> Programmer verifies that business logic of a system feature works.

- <!-- .element: class="fragment" --> Test the whole system (except UI)
    - Inbound API (REST, msg)
    - Prevent problems with _UI tests_
    - More than _unit tests_
- <!-- .element: class="fragment" --> Mock outbound system interfaces
    - Our system only tested


Notes:
- Exclude UI to get rid of the problems with UI tests,  
  but otherwise cover the whole app (or the most possible)
- UI tests problems:
    - complexity (asynchronicity)
    - fragility (dependence on details)
    - other tests needed (no UX)
    - tooling clumsy


^^^^

## Generic test approach

1. Setup DB state
2. Execute system feature under test
3. Verify system response
4. Verify DB state
5. Verify outbound requests


^^^^

<!-- .slide: data-transition="fade" -->

## Common anti-patterns
#### Setup DB state

- <!-- .element: class="fragment" --> DB reused
    - <!-- .element: class="fragment" --> Setup DB once at start of test suite  
    - <!-- .element: class="fragment" --> &Implies; No defined setup state **(!)**
    - <!-- .element: class="fragment" --> &Implies; Tests inter-dependent. **Don't!**


Notes:
- DB reused, Tests inter-dependent:
    - False positives: non-deterministic tests
    - Test or test suites not repeatable
    - &Implies; Fix or delete


^^

<!-- .slide: data-transition="fade" -->

## Common anti-patterns
#### Setup DB state

- <!-- .element: class="fragment" --> DB setup reused
    - <!-- .element: class="fragment" --> Single setup dataset shared between tests
    - <!-- .element: class="fragment" --> &Implies; Dataset not maintained **(!)**


^^

<!-- .slide: data-transition="fade" -->

## Common anti-patterns
#### Verify system response <br> Verify DB state <br> Verify outbound requests

- <!-- .element: class="fragment" --> Partial verification
    - <!-- .element: class="fragment" --> Arbitrary subset of data
    - <!-- .element: class="fragment" --> Assertions on single values
    - <!-- .element: class="fragment" --> &Implies; False negatives. **Don't!**


Notes:
- False negative:
    - (Ineffective test)
    - Undetected until bug reported from  
      other test (type) or users (!!)


^^

<!-- .slide: data-transition="fade" -->

## Common anti-patterns
#### Setup DB state <br> Verify DB state

- <!-- .element: class="fragment" --> App code reused 
    - <!-- .element: class="fragment" --> Use repo to setup / verify DB
    - <!-- .element: class="fragment" --> Or app service - even worse
    - <!-- .element: class="fragment" --> Bug in app &Implies; bug in test? **(!)**
    - <!-- .element: class="fragment" --> &Implies; False negatives. **Don't!**


^^^^

## DB setup & verification

- <!-- .element: class="fragment" --> Tool: Light Air
- <!-- .element: class="fragment" --> Since 2012 (DbUnit, Unitils 2007&rarr;12)
- <!-- .element: class="fragment" --> 7 (+3) projects myself
- <!-- .element: class="fragment" --> Postgres, H2, HSQL, Oracle


Notes:
- I am the author, others contributed
- 7 projects Light Air, 3 project before that DbUnit / Unitils
- When I come to a project with backend in Java
    - Light Air becomes the tool to be used


^^^^

<!-- .slide: data-transition="fade" -->

## DB setup: algorithm

1. <!-- .element: class="fragment" --> `Delete` all rows
2. <!-- .element: class="fragment" style="margin-bottom: 5rem;" --> `Insert` data rows for the test


<!-- .element: class="fragment" --> &Implies; Specify only data, declaratively


^^

<!-- .slide: data-transition="fade" -->

## DB setup: data

- <!-- .element: class="fragment" --> Dataset: flat XML format

```xml
<dataset>
  <orders id="1" number="1234-567"/>
  <line_items id="1" order_id="1" quantity="10"/>
  <line_items id="2" order_id="1" quantity="30"/>
</dataset>
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> Elements = DB tables
- <!-- .element: class="fragment" --> Attributes = DB columns
- <!-- .element: class="fragment" --> In setup, performs:
    1. <!-- .element: class="fragment" --> `delete from line_items, orders`
    2. <!-- .element: class="fragment" --> `insert into orders, line_items`


Notes:
- Empty element: deleted, but not inserted


^^

<!-- .slide: data-transition="fade" -->

## DB setup: benefits

- <!-- .element: class="fragment" --> Tests repeatable, independent
- <!-- .element: class="fragment" --> Setup data specific to the test
    - &Implies; maintainable
- <!-- .element: class="fragment" --> No test code (provided by the tool):
    - &Implies; no bugs in test code


^^^^

<!-- .slide: data-transition="fade" -->

## DB verify: algorithm

1. <!-- .element: class="fragment" --> `Select` all rows from the tables
2. <!-- .element: class="fragment" style="margin-bottom: 5rem;" --> Compare **all** data selected with dataset


<!-- .element: class="fragment" --> &Implies; Specify only data, declaratively


^^

<!-- .slide: data-transition="fade" -->

## DB verify: data

- <!-- .element: class="fragment" --> The same dataset format as for setup

```xml
<dataset>
  <orders id="1" number="1234-567"/>
  <line_items id="1" order_id="1" quantity="10"/>
  <line_items id="2" order_id="1" quantity="30"/>
</dataset>
```
<!-- .element: class="fragment" -->

- <!-- .element: class="fragment" --> In verify, performs:
    1. <!-- .element: class="fragment" --> `select from orders, line_items`
    2. <!-- .element: class="fragment" --> Verify 1 row in `orders` & values match
    3. <!-- .element: class="fragment" --> Verify 2 rows in `line_items` & values match


^^

<!-- .slide: data-transition="fade" -->

## DB verify: benefits

- <!-- .element: class="fragment" --> Extremely easy to verify many data
- <!-- .element: class="fragment" --> &Implies; Can cover **the WHOLE** DB state
    - &Implies; full (not partial) verification
- <!-- .element: class="fragment" --> No test code (provided by the tool):
    - &Implies; no bugs in test code
- <!-- .element: class="fragment" --> More tests get written


^^^^

<!-- .slide: data-transition="fade" -->

## Tokens

Special values

- <!-- .element: class="fragment" --> `@null`: explicit null value
- <!-- .element: class="fragment" --> `@any`: any non-null value
- <!-- .element: class="fragment" --> `@date`: now, precision date (last midnight)
- <!-- .element: class="fragment" --> `@time`: now, precision time
- <!-- .element: class="fragment" --> `@timestamp`: now, precision timestamp

```xml
<dataset> <!-- verify -->
  <orders id="@any" number="1234-567"
          comment="@null" created="@timestamp"/>
</dataset>
```
<!-- .element: class="fragment" -->


^^

<!-- .slide: data-transition="fade" -->

## Durations

Modify temporal tokens with + / -  
and ISO 8601 duration

- <!-- .element: class="fragment" --> `@date` = last midnight
- <!-- .element: class="fragment" --> `@date+P1D` = first following midnight
- <!-- .element: class="fragment" --> `@date-P1M` = midnight one month before last
- <!-- .element: class="fragment" --> `@date+PT12H` = noon today
- <!-- .element: class="fragment" --> `@timestamp+PT1H` = one hour in the future
- <!-- .element: class="fragment" --> `@timestamp-P2Y3M4DT5H6M7S` = ?


^^

<!-- .slide: data-transition="fade" -->

## Variables

- <!-- .element: class="fragment" --> Start a value with `$` to define a var
- <!-- .element: class="fragment" --> First occurence: reads value from DB
- <!-- .element: class="fragment" --> Following occurences: verify value matches
- <!-- .element: class="fragment" --> Verify consistency of generated values
    - <!-- .element: class="fragment" --> Surrogate foreign keys

```xml
<dataset> <!-- verify -->
  <order id="$orderId" number="1234-567" />
  <line_item id="@any" order_id="$orderId" quantity="10" />
  <line_item id="@any" order_id="$orderId" quantity="30" />
</dataset>
```
<!-- .element: class="fragment" -->


^^

<!-- .slide: data-transition="fade" -->

## Auto-generated values

- <!-- .element: class="fragment" --> Token `@auto`
- <!-- .element: class="fragment" --> Guaranteed unique (exc. booleans)
    - <!-- .element: class="fragment" --> &Implies; Test mapping of attributes
- <!-- .element: class="fragment" --> Indicate particular value not relevant

```xml
<dataset> <!-- setup -->
  <order id="@auto" number="@auto" created="@auto"/>
  <order id="@auto" number="@auto" created="@auto"/>
  <order id="@auto" number="@auto" created="@auto"/>
</dataset>
```
<!-- .element: class="fragment" -->


Notes:
- Setup: value required, but not relevant


^^^^

## XSD

- <!-- .element: class="fragment" --> Read DB structure & generate XSD
- <!-- .element: class="fragment" --> Maven plugin, easily incorporate into build
- <!-- .element: class="fragment" --> IDE (editor) suggests tables, columns, values
- <!-- .element: class="fragment" --> Very easy editing of dataset files


^^

## XSD (Clojure)

```Clojure
(ns lein-run (:gen-class)
  (:require [mount.core :as mount])
  (:import net.sf.lightair.Api))
(defn db-update []
  (start-embedded-db)
  (try
    (Api/generateXsd)
    (finally
      (mount/stop))))
```

```Clojure
(defproject ...
  :aliases {"db-update" ["run" "-m" "lein-run/db-update"]}
)
```

^^^^

<!-- .slide: data-transition="fade" -->

## Writing a test (Java)

```Java
package it.customer.create;
import net.sf.lightair.LightAir;
import net.sf.lightair.annotation.Setup;
import net.sf.lightair.annotation.Verify;

@RunWith(LightAir.class)
@Setup
public class CreateCustomerIT {
	@Test
	@Verify
	public void full() {
	    // ...invoke API...
	}
}
```


Notes:
- Package name convention: it / entity / action
- @RunWith(LightAir.class)
- @Setup, @Verify
    - On method or class
- DB taken care of
    - Only invoke the app
    - And verify the response
    - If integrations, verify outbound requests


^^

<!-- .slide: data-transition="fade" -->

## Default dataset resolution (Java)

1. &lt;test method name>(-verify).xml
2. &lt;test class name>(-verify).xml


^^

<!-- .slide: data-transition="fade" -->

## Custom dataset resolution (Java)

```Java
@Setup("CreateCustomerIT.xml")
@Verify("CreateCustomerIT.xml") // verify no change in DB
@Verify("shared-verify.xml")
@Setup({"this-verify.xml", "that-verify.xml"})
@Setup.List({
        @Setup({"this.xml", "that.xml"}),
        @Setup               // ...and default resolution
})
```


^^

## Test support functions (Clojure)

```Clojure
(ns lightair
  (:require [mount.core :refer [defstate]])
  (:import net.sf.lightair.Api))

(defstate lightair
          :start (Api/initialize
                   "dev/resources/light-air.properties")
          :stop (Api/shutdown))
```

```Clojure
(defn- process-files [prefix files]
  (vec (map #(str "test/it/" prefix %1 ".xml") files)))

(defn db-setup [prefix & files]
  (Api/setup {Api/DEFAULT_PROFILE,
              (process-files prefix files)}))

(defn db-verify [prefix & files]
  (Api/verify {Api/DEFAULT_PROFILE,
               (process-files prefix files)}))
```


^^

<!-- .slide: data-transition="fade" -->

## Writing a test (Clojure)

```Clojure
(ns it.customer.create.customer-create-test
  (:require [clojure.test :refer [deftest]]
            [lightair :refer :all]))

(def ^:private prefix "customer/create/")

(deftest customer-create-full
  (db-setup prefix "../../users" "setup")
  ; ...invoke API...
  (db-verify prefix "full-verify"))
```


^^^^

## Invoking RESTful APIs (Java)

- <!-- .element: class="fragment" --> Tool: REST Assured
- <!-- .element: class="fragment" --> Promotes partial response verification :(
- <!-- .element: class="fragment" --> Wrapper to support full verification:
    1. Load JSON from file
    2. Re-format file _and_ response
    3. Compare as strings
- <!-- .element: class="fragment" --> Support replacing dynamic values


^^

## Invoking RESTful APIs (Clojure)

- <!-- .element: class="fragment" --> Call the handler
- <!-- .element: class="fragment" --> Functions to support full verification:
    1. Load JSON from file
    2. Re-format file _and_ response
    3. Compare as strings


^^^^

<!-- .slide: data-transition="fade" -->

## Things to test

- <!-- .element: class="fragment" --> Ok:
    <span class="fragment">full,</span>
    <span class="fragment">minimal,</span>
    <span class="fragment">search variants</span>
- <!-- .element: class="fragment" --> Validations:
    <span class="fragment">empty,</span>
    <span class="fragment">long,</span>
    <span class="fragment">short,</span>
    <span class="fragment">duplicate,</span>
    <span class="fragment">invalid</span>
- <!-- .element: class="fragment" --> Errors:
    <span class="fragment">not found,</span>
    <span class="fragment">conflict,</span>
    <span class="fragment">unauthorized</span>
- <!-- .element: class="fragment" --> HTTP:
    <span class="fragment">response codes,</span>
    <span class="fragment">headers</span>


^^

<!-- .slide: data-transition="fade" -->

## Programmer's workflow (Java)

- <!-- .element: class="fragment" --> App restart
    - small app: 15 sec (inc. Postgres)
    - medium app (25 tables): 20 sec
- <!-- .element: class="fragment" --> Single test method: 3 sec (JUnit: 1.5 sec)
- <!-- .element: class="fragment" --> Test class (7 tests): 3 sec (JUnit 1.9 sec)
- <!-- .element: class="fragment" --> Entity (30 tests): 5 sec (JUnit 2.5 sec)
- <!-- .element: class="fragment" --> All tests (medium app, 650 tests): 1.5 min


^^

<!-- .slide: data-transition="fade" -->

## Programmer's workflow (Clojure)

Medium app (20 tables)

- <!-- .element: class="fragment" --> App restart: 300 ms (inc. Postgres)
- <!-- .element: class="fragment" --> Single test method: 30 (< 100) ms
- <!-- .element: class="fragment" --> All tests (1500 assertions): 12 sec


^^^^

<!-- .slide: data-transition="fade" -->

## Test strategy
### ...of the past

- <!-- .element: class="fragment" --> Testing pyramid
    - (low) unit > system > UI (high)
- <!-- .element: class="fragment" --> Balance time (write & run) vs. benefit


^^

<!-- .slide: data-transition="fade" -->

## Unit / service tests

- <!-- .element: class="fragment" --> Missing API aspects:
    - HTTP URLs, params, headers, response codes
- <!-- .element: class="fragment" --> Service throws Exception, but HTTP response?
- <!-- .element: class="fragment" --> Time **NOT** an excuse anymore
    - System tests easy to write AND fast to run
- <!-- .element: class="fragment" --> Unit tests best fit:
    - Complex algorithms with many variants


^^

<!-- .slide: data-transition="fade" -->

## System tests FTW

- <!-- .element: class="fragment" --> Full app stack tested
- <!-- .element: class="fragment" --> Full response / DB verification
- <!-- .element: class="fragment" --> Tools encourage proper test design
- <!-- .element: class="fragment" --> Tests simple AND easy to write
    - <!-- .element: class="fragment" --> More data gets verified
    - <!-- .element: class="fragment" --> More tests get written
- <!-- .element: class="fragment" --> Live documentation


Notes:
- Proper test design: e.g. separate DB setups


^^^^

## Conclusion

- <!-- .element: class="fragment" --> Testing pyramid is dead!
- <!-- .element: class="fragment" --> Cover everything by system tests
- <!-- .element: class="fragment" --> Have happy-day UI tests + manual
- <!-- .element: class="fragment" --> Algorithms: unit tests or Cucumber
- <!-- .element: class="fragment" --> (&Implies; _Testing double-cone_)


Notes:
- Testing pyramid is outdated idea, for years already


^^^^

## References

- Light Air: http://lightair.sourceforge.net/
- REST Assured: http://rest-assured.io/
- Model project (Java, Spring):  
    https://github.com/ivos/lightair-spring-sample
    - All tools put together
    - Real-life features covered with system tests
- Model project (Clojure):
    https://github.com/ivos/clojure-rest-jdbc-backend
- These slides:  
    https://ivos.github.io/testing-db-apps-slides/
