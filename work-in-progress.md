# recent song list for klipse (kind of works)

## Include `clojure.test`

We are writing our tests with the `clojure.test` library so we need to include that first

```eval-clojure
(ns recent-song-list
  (:require [cljs.test
              :refer-macros
              [deftest is testing run-tests run-all-tests]]))
```

> ####Bug::Put namespace & require in a klipse preamble, as the `:refer-macros` is specific to Clojurescript.


### Run Tests

At any time we can use the `run-tests` function to get a report back on all of the tests in our current namespace (`recent-song-list`)

```eval-clojure
(run-tests)
```

> ####Warning::Re-run tests
> Is there a way to re-run the tests without having to change the `(run-tests)` expression.  We could put it in a loop, but then its running the tests all the time and that may use up resources (worth a try though).

The following will run tests every 30 seconds

<pre><code class="lang-eval-clojure" data-loop-msec="30000">
(run-tests)
</code></pre>

## Define a recent song list

```eval-clojure
(def recent-songs [])
```

> ####Note::Write a test to check a song-list exists
> Write a test to see if a recent song list exists.
>
> This is an opportunity to think about what kind of data structure you want to use to hold your recent song list.
```eval-clojure
(deftest song-list-exists-test
  (testing "Does a recent song list exist"
    (is
     (vector? recent-songs))))
```

<!--sec data-title="Suggested test..." data-id="answer001" data-collapse=true ces-->

A simple test that checks for a `recent-songs` list, checking

```clojure
(deftest song-list-exists-test
  (testing "Does a recent song list exist"
    (is
     (vector? recent-songs))))
```

<!--endsec-->

> ####Note::Write a test to check the song-list is empty
> The recent song list should be empty to start with.
```eval-clojure
```clojure
(deftest song-list-empty-test
  (testing "Is song list empty if we havent added any songs"
    (is
     (= [] recent-songs))))
```

<!--sec data-title="Suggested test..." data-id="answer002" data-collapse=true ces-->

A simple test that compares an empty vector with the value of `recent-songs`

```clojure
(deftest song-list-empty-test
  (testing "Is song list empty if we havent added any songs"
    (is
     (= [] recent-songs))))
```
Here is the same test using the `empty?` function instead of the `=` function.

```clojure
(deftest song-list-empty-test-2
  (testing "Is song list empty if we havent added any songs"
    (is
     (empty? recent-songs))))
```

> ####Hint::
> You could use either of these tests to replace the song list exists test, as these tests would fail if the song list did not exist.

<!--endsec-->


> #### Note::Write a test to add a song to the list


<!--sec data-title="Suggested test..." data-id="answer003" data-collapse=true ces-->

```clojure
(deftest adding-songs-test

  (testing "add song returns a song list with entries"
    (is
     (not (empty?
           (add-song "Barry Manilow - Love on the rocks" recent-songs)))))

  (testing "add multiple song returns a song list with entries"
    (is
     (not (empty?
           (->> recent-songs
             (add-song "Barry Manilow - Love on the rocks")
             (add-song "Phil Colins - Sususudio" )))))))

```

<!--endsec-->




<!--sec data-title="Suggested Code Solution..." data-id="answer009" data-collapse=true ces-->

This suggested solution defines our recent songs as an empty vector.

The `add-song` function takes the name of a song and the song list to which it will be added.

A Thread-last macro `->>` is used to pass the song list over two functions.

The `song-list` is first passed to the `remove` expression as its last argument.  This expression will remove any occurrence of the new song we want to add from the `song-list`.

The results of the `remove` expression are then passed to the `cons` expressoin as its last argument.  The `cons` expression simply adds the new song to the start of the list, making it the most recent song.

```clojure
(def recent-songs [])

(defn add-song [song song-list]
  (->> song-list
       (remove #(= song %))
       (cons song)))
```
<!--endsec-->



## Some more stuff on tests


==== Discussion

The preceding example only scratches the surface of what `clojure.test` provides for unit testing. Let's take a bottom-up look at its other
features.

First, you can improve reporting when an assertion fails by providing a second argument that explains what the assertion is intended to
test. When you run this test, you will see an extended description of how the code was expected to behave:

```clojure
(is (= (capitalize-entries {:office "space"} :office) {})
    "The employee's office entry should be capitalized.")
;; -> false
;; * out*
;; FAIL in clojure.lang.PersistentList$EmptyList@1 (NO_SOURCE_FILE:1)
;; The employee's office entry should be capitalized.
;; expected: (= (capitalize-entries {:office "space"} :office) {})
;;   actual: (not (= {:office "Space"} {}))

```

For testing a function like +capitalize-entries+ thoroughly, several use cases need to be considered. To more concisely test numerous similar cases, use the +clojure.test/are+ macro:


```clojure
(deftest test-capitalize-entries
  (let [employee {:last-name "smith"
                  :job-title "engineer"
                  :level 5
                  :office "seattle"}]
    (are [ks m] (= (apply capitalize-entries employee ks) m)
         [] employee
         [:not-a-key] employee
         [:job-title] {:job-title "Engineer"
                       :last-name "smith"
                       :level 5
                       :office "seattle"}
         [:last-name :office] {:last-name "Smith"
                               :office "Seattle"
                               :level 5
                               :job-title "engineer"})))
```

The first two parameters to +are+ set up a testing pattern: given a sequence of keys +ks+ and a map +m+, call +capitalize-entries+ for those keys on the original +employee+ map and assert that the return value equals +m+.

Writing out multiple use cases in a declarative syntax makes it easier to catch errors and untreated edge cases, such as the +NullPointerException+ that will be thrown for the `[:not-a-key] employee` assertion pair in the preceding test.(((truthiness)))


## Running tests in leiningen

----
$ lein test

lein test com.example.core-test
lein test com.example.util-test

Ran 10 tests containing 20 assertions.
0 failures, 0 errors.
----

To limit the scope of tests Leiningen runs, use the +:only+ option,
followed by a fully qualified namespace or function name:

[source,shell-session]
----
# To run an entire namespace
$ lein test :only com.example.core-test

lein test com.example.core-test

Ran 5 tests containing 10 assertions.
0 failures, 0 errors.

# To run one specific test
$ lein test :only com.example.core-test/test-capitalize-entries

lein test com.example.core-test

Ran 1 tests containing 2 assertions.
0 failures, 0 errors.
----


Unlike testing frameworks for other popular dynamic languages, Clojure's built-in assertions are minimal and simple. The +is+ and +are+ macros check test expressions for "truthiness" (i.e., that those expressions return neither +false+ nor +nil+, in which case they pass). Beyond this, you can also check for +thrown?+ or +thrown-with-msg?+ to test that a certain +java.lang.Throwable+ (error or exception) is
expected:

[source,clojure]
----
(is (thrown? IndexOutOfBoundsException (nth [] 1)))
----

Above the level of individual assertions, +clojure.test+ also provides
facilities for calling functions before or after tests run. In the
+test-capitalize-entries+ test, we defined an ad hoc +employee+ map for
testing, but you could also read in external data to be shared across
multiple tests by registering a data-loading function as a "fixture."
The +clojure.test/use-fixtures+ multimethod allows registering Clojure
functions to be called either before or after each test, or before or
after an entire namespace's test suite. The following example defines
and registers three fixture functions:

[source,clojure]
----
(require '[clojure.edn :as edn])

(def test-data (atom nil))

;; Assuming you have a test-data.edn file...
(defn load-data "Read a Clojure map from test data in a file."
  [test-fn]
  (reset! test-data (edn/read-string (slurp "test-data.edn")))
  (test-fn))

(defn add-test-id "Add a unique id to the data before each test."
  [test-fn]
  (swap! test-data assoc :id (java.util.UUID/randomUUID))
  (test-fn))

(defn inc-count "Increment a counter in the data after each test runs."
  [test-fn]
  (test-fn)
  (swap! test-data update-in [:count] (fnil inc 0)))

(use-fixtures :once load-data)
(use-fixtures :each add-test-id inc-count)

;; Tests...
----

You can think about fixture functions as forming a pipeline through
which each test is passed as a parameter, which we called +test-fn+ in
the preceding example. Take +inc-count+, for example. It is the job of this
fixture to invoke the +test-fn+ function, continuing the pipeline, and
afterward, to increment a count (i.e., "do some work"). Each fixture
decides whether to invoke +test-fn+ before or after its own work
(compare the +add-test-id+ function with the +inc-count+ function),
while the +clojure.test/use-fixtures+ multimethod controls whether
each registered fixture function is run only once for all tests in a
namespace or once for each test.

Finally, with a firm understanding of how to develop individual
Clojure test suites, it is important to consider how you organize and
run those suites as part of your project's build. Although Clojure
allows defining tests for functions anywhere in your code base, you
should keep your testing code in a separate directory that is only
added to the JVM classpath when needed (e.g., during development and
testing). It is conventional to name your test namespaces after the
namespaces they test, so that a file located at
_<project-root>/src/com/example/core.clj_ with namespace
+com.example.core+ has a corresponding test file at
_<project-root>/test/com/example/core_test.clj_ with namespace
+com.example.core-test+. To control the location of your source and
test directories and their inclusion on the JVM classpath, you should
use a build tool like link:http://leiningen.org/[Leiningen] or
link:http://maven.apache.org/[Maven] to organize your project.

In Leiningen, the default directory for your tests is a top-level
_<project-root>/test_ folder, and you can run your project's tests with
+lein test+ at the command line. Without any additional arguments, the
+lein test+ command will execute all of the tests in a project:

[source,shell-session]



==== See Also

* The +clojure.test+ http://bit.ly/clj-test-api[API documentation] contains full information on the unit-testing framework.
* If you are instead using Maven, use   https://github.com/talios/clojure-maven-plugin[+clojure-maven-plugin+]
  to run Clojure tests. This plug-in will incorporate your Clojure
  tests located in the Maven standard _src/test/clojure_ directory as
  part of the +test+ phase in the Maven build life cycle. You can
  optionally use the plug-in's +clojure:test-with-junit+ goal to
  produce JUnit-style reporting output for your Clojure test runs.
