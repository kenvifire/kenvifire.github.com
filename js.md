###JavaScript笔记
##Comment
<pre><code>
// Single Line Comments
// Yet another single line comment
// More single line comment!!!

/*
Javascript also supports
multi
line
comments!
*/
</code></pre>

####Variables
1. Undefined and null

>1.undefined 在一些老的浏览器里不是关键字

>2.用`===`来进行更严格的检查，而不是用`==`

2.Scope  and Variable shadowing

3.Variable Hoisting 


###Data Type

#####String

#####Numbers

1、parsing numbers from strings

>parseInt()
>parseFloat()

2、Number Comparison

	logData(typeof Math); // object
	logData(typeof Number); // function

3、Number Object methods

3.1、Demos

	num.toExponential(n)//n is in [0,20]
	num.toFixed(n)// n is in [0,20]
	num.toPrecision(n)// n is in [0,21]
	num.toString(radix)
	num.toLocaleString()
	num.valueOf()//// Returns the primitive value of a number

3.2、Differences:

	typeof number.toPrecision();//string
	typeof number.toPrecision();//string

	typeof number.valueOf();//number

3.3、emphasis

	// - If the number literal has no exponent
	// and no decimal point, then add a space
	// before the dot that precedes the method call.
	logData(123 .toString()); // 123
	// - Add a decimal point before the dot
	// that precedes the method call.
	logData(123..toString()); // 123
	

	

###4、Conditional Statements

4.1、if...else

4.2、switch

###5、Operators

5.1、conditional ternary operator

	result=='test'?true:false

5.2、 comma operator and separator

	5.2.1  comma operator

	The comma operator is a simple one that evaluates
	both of its operands (left-to-right) and returns the value of the second operand.

	5.2.2 separator
	
	In these cases, the comma used is a
	"comma separator" NOT a "comma operator".	

	var x, y;
	var arr = [1, 2, false, 'foo'];
	var obj = {
		foo: 'bar',
		baz: 123.456
	};

5.3、delete operator
>delete operator can **delete a property of an object or an array element**.

5.3.1 delte global variable

	myVar = 10;
	logData(myVar); // 10
	logData(delete myVar); // true
	logData(typeof myVar); // undefined

	**cann't delete variables defined with the 'var' keyword.**

	var iVar = 20;
	logData(iVar); // 20
	logData(delete iVar); // false !! CANT delete!
	logData(iVar); // 20
	
	
5.3.2 Deleting an object property

	var myOb = {foo: 'bar', baz: 'bak'};
	logData(myOb.foo); // bar
	logData(delete myOb.foo); // true
	logData(myOb.foo); // undefined
	
>**Some properties of predefined objects cannot be deleted**

	logData(delete Number.POSITIVE_INFINITY); // false
	logData(delete Math.PI); // false

5.3.3  Deleting array elements
	
	var arr = ['foo', 'bar'];
	logData(arr);
	logData(delete arr[0]); // trye
	logData(arr);
	// But there's a problem...
	logData(arr.length); // 2 !! length didnt change :O

5.4、in,instance of and typeof Operators

5.4.1 typeof
	
	someVar = false;
	logData(typeof someVar); // boolean
	logData(typeof 123.456); // number
	logData(typeof {foo: 'bar'}); // object
	logData(typeof nonExistantVariable); // undefined

5.4.2 in

>**This operator can be used to check whether a
	string or a numeric property exists in an
	object or not.**	

	var myOb = {foo: 'bar', 2: 'numeric'};
	logData('foo' in myOb); // true
	logData(2 in myOb); // true
	
	logData('bar' in myOb); // false

>**`in` can also be used to find whether an
	element with a particular index exists
	in an array or not.**
	
	var arr = ['hello', 'world', false, 123.456];
	logData(0 in arr); // true
	logData(2 in arr); // true
	logData(5 in arr); // false

5.4.3 instanceof

**The `instanceof` operator:
This operator basically tells us if a
specified object is an instance of the
specified Object Type (Constructor)
or not.
This means, that the operator checks
whether an object has the specified
constructor's `prototype` property
in its prototype chain or not.**

5.5 void operator

**The `void` operator basically, converts any
	value to undefined.**

	logData(void 123.456); // undefined

###Functions


###Data Structures

######Arrays

######Objects

###Loops


###OOP


###Extras

#####JSON

#####Regexp




