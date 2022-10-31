* REgex: #"something$"
* Types of things: function, value, keyword, namespace
* nil, true, false

* Keywords always with leading : - :alpha
* Can be in a namespace: :release/alpha

* Types of collection:
  * List: '(1 2 3) -> Linked list! Grows from head
  * Vector: [1 2 3]
  * Set: #{1 2 3}
  * Map: {:a 1, :b 2}  ; Note keywords

* () is an invocation; fn required in position 0

* Literals: 'x -> symbol "x" without dereferencing
	* List: '(2 2 2)

* *1 == last REPL result; *2 == second last, *3 - third last

* Load a library: (require '[clojure.repl :refer :all])

* (def x 7) - creates a "var"; should be a const or fn for production
  code -> #'user/x is literal representation.

* Print stuff: prn, pr -> machine readable, print, println -> translate \n, specials

* Define functions w/ defn: (defn greet [name] (str "Hello, " name) )
* STring contatenation: (str "some" "stuff")
* Fn overloading == "arities". Define multiple arities (arity means "number of arguements") w/ same def:
  (defn messenger
  	([]								(messenger "Hello world!"))
  	([message]				(println message)))
* Variadic = variable arglist. Like defn, but variable parms collected in sequence in identifier at the end:
	(defn variadic [a b & c] 
		(println a b (cond (nil? c) "" c))  ; works because c is printlned, and that prints the string repr of the seq
* Lambda/anonymous func: (fn [parms] (print parms))
	* Short form w/ # - #(print %1 %2 %3 %&) - no need for vector of args cuz %1, etc. Can use "%" for just one parm.
* let - Run some code with a sub-scope:
	(def a 10)
	(def b 20)
	(print (+ a b))     ; prints 30

	(let [
		a 1 
		b 2
	] (print (+ a b)))  ; prints 3

	(print (+ a b))     ; prints 30

* Closures: return an anonymous func inside a defn that refers to a value in scope in the defn; stores
	that value for future use.
	(defn messenger-builder [in_val] 
		(fn [another] (
			println "in: " in_val ", another: " another)))

* Useful functions:
	* (cons a b seq) - build sequence of parms + seq
	* (apply myfunc 1 2 [3 4 5]) - kinda unpack; call (myfunc 1 2 3 4 5)
	* first, second, next, last - access to collection elements

* Java calls:

Task										Java											Clojure	
Instantiation 					new Widget("foo") 				(Widget. "foo")
Instance method					rnd.nextInt()							(.nextInt rnd)
Instance field					object.field							(.-field object)
Static method						Math.sqrt(25)							(Math/sqrt 25)
Static field						Math.prints								Math/PI


* Vectors: use "get" to access.
	(assert (= (get [1 2 3] 0) 1))    ; invalid index returns nil
		* Can invoke directly: ([2 1 0] 2)   ; 0
	(count [1 2 3]) ; 3
	(vector 1 2 3) ; programmatic creation of [1 2 3]

* Lists -> linked list
	* Grow from head
	* No indexing; work w/ first (duh), next (all after first)
  * conj will add, but from the front

* Sets
  * Add with conj
  * Remove with disj - non-members won't bork
  * Check membership w/ contains?
     * Invoke directly: returns value searched if present; nil else
  * join sets with into - (into setA setB)

* Maps
  * Define with {} - (def mymap {1 2, 3 4, 5 6})
  * Add new with assoc - (assoc mymap 7 8)
  * Remove with dissoc - (dissoc mymap 7)
  * Find by key - (get mymap 7)
  	* OR can invoke map directly: (mymap 7)
  	* Can provide a default to either get or invocation as extra parm
  * find function - returns the pair as a vector. (find {1 2 3 4} 3)    ; returns [3 4]
  * zipmap - zip two sequences/vectors into a map: (zipmap(zipmap [2 4 6 8] [1 2 3 4]) -> {2 1, 4 2, 6 3, 8 4}
  * Compbine with merge; dups mean second map wins, or you can provide resolver w/ merge-with
  * Can use keywords as map keys: (def a {:dog "milly" :car "tesla"})
  	* In this case, can invoke the keyword w/ map as argument:
  	(:dog a) -> "milly"
  * Use get-in to traverse into a nested map structure; also assoc-in or update-in to write
  * Also, records: 
		;; Define a record structure
		(defrecord Person [first-name last-name age occupation])

		;; Positional constructor - generated
		(def kelly (->Person "Kelly" "Keen" 32 "Programmer"))

		;; Map constructor - generated
		(def kelly (map->Person
		             {:first-name "Kelly"
		              :last-name "Keen"
		              :age 32
		              :occupation "Programmer"}))

* If statements
	* Ternary style: (str "2 is " (if (even? 2) "even" "odd"))
	* Or no else: (if (true? false) "Nope; not coming here")  ; identical to (when true "stuff")
* Only false and nil are false; everything else is true
* If--then-else ladder-like thing with cond. Note straight pairs of condition, if-matched.
	(cond
		(< x 2) "Less than two"
		(< x 5) "Less than five"
		(< x 10) "Less than ten"
		:else "greater than ten"    ; by convention - works because :else evaluates to true, like all keywords
	)
* case - match a value in constant time; value has to be compile-time literal, and input values have to be predictable - you get an IllegalArgumentException if the input doesn't match.
	(case x
		5 "x is 5"
		10 " x is 10"
	)