---
layout: post
title: "Intro to Test-Driven Development in Python"
permalink: /resources/tutorials/fizzbuzz-py
---

To complete this tutorial, you will need to have the following installed:
- Python
- Pip
- Git
- A text editor
- A terminal or command prompt (we will be working mainly from the terminal, if
	you are not comfortable using yours, you may want to complete a [Command Line Crash Course](https://learnpythonthehardway.org/python3/appendixa.html)
	before you continue).

Think you are missing something? [Check/Install here](https://github.com/fouralarmfire/square-one/blob/master/tutorials/tdd-setup.md).

## Task:
Using [TDD](https://www.agilealliance.org/glossary/tdd), write a program which
will help you cheat at the drinking game fizzbuzz.

#### Rules:
- When the given number is divisible by 3, say fizz
- When the given number is divisible by 5, say buzz
- When the given number is divisible by 3 and 5, say fizzbuzz!
- When the given number does not fit any of the other rules, print the number

```sh
$ python fizzbuzz.py 3
fizz

$ python fizzbuzz.py 5
buzz

$ python fizzbuzz.py 15
fizzbuzz!

$ python fizzbuzz.py 7
7 :(
```

## Part 1: Project setup

**Steps**:
1. Open Terminal (or iTerm or whatever else you like to get a command prompt) and create a new directory.
    Then change into that directory and initialise a new git repository _(New to Git? See [this guide](/resources/tutorials/git).)_:
 
    _nb: Do not just paste in the `<CAPS IN ARROWS>`, they are placeholders._

	```sh
	cd ~
	mkdir -p workspace/fizzbuzz # or use another location if you already have a place to store your projects
	cd workspace/fizzbuzz
	git init
	# create your repo online
	git remote add origin <URL OF YOUR REPO ONLINE>
	echo "tdd exercise in python" > README.md
	git add README.md
	git commit -m "readme.md"
	git push -u origin main
	```

## Part 2: Our first test

Test Driven Development follows a simple pattern: `red -> green -> refactor`.
In reality, this works as follows:
1. Write a test which would pass if the code was implemented correctly.
1. See it fail.
1. Write _just_ enough code to make the test pass.
1. See the test pass.
1. Look over the code and think of ways to improve what you have. Is there any
repetition? Can an algorithm be made more efficient?

Writing code this way has 4 benefits: 1) by only writing
what you need to achieve basic tasks, you end up writing less code, all of which
is used (no ghost functions which you have little memory of); 2) every single
function is tested, which makes adding more or changing bits a breeze as you will
find out immediately what you may have broken; 3) nicely structured tests makes
it easy for others looking at your project to figure out what your code is
supposed to do (a good way to get contributors); and 4) because of 1 and 2, you have
complete confidence that your code does what it is supposed to do.

![alt text](http://turnoff.us/image/en/tdd-vs-ftt.png)
_[turnoff.us](http://turnoff.us/geek/tdd-vs-ftt/)_

So let's get going with our first test:

**Steps**:

1. We are going to use a package called [`pytest`](https://docs.pytest.org/en/latest/) to run our
  tests. To check whether you have `pytest` installed, run `pytest --version`. If that doesn't return
  "This is pytest version etc", then follow the installation instructions [here](https://docs.pytest.org/en/latest/getting-started.html).

1. Create a new file called `fizzbuzz_test.py`.

1. Open the project in your text editor, and write the following to it:

	```py
	def test_that_number_is_divisible_by_three():
	    return
	```

	So what have we done here?
	You'll notice that we haven't just written `test_that_everything_works`. TDD is about ensuring
	that you write your code incrementally by breaking down your problem into small
	chunks and tackling each one at a time. Right now we are writing a pretty simple
	game, so you may be thinking TDD is overkill, but when it comes to a large project
	or a very complex problem which you can't possibly envisage how it will turn out,
	TDD is a very useful discipline to learn.
	
	So let's read our test.
	We have defined the function which is going to test the first bit of our code.
	Right now our function doesn't test anything and just returns nothing.
	We could have called this function anything, but the descriptive name is for our benefit.
	The only rule is that it has to start with `test_`.

1. Run the test by typing `pytest` in the terminal.
	This should show you a happy green line with `1 passed in 0.05 seconds` or something.
	Of course we are not actually testing anything: very little can go wrong with returning nothing.

1. So let's write our first real test. We have to state what we expect to happen when our code runs.
	We have not written any code yet, but we have explained what we want
	in our by our test function name, so we know vaguely what it should look like. Alter your test function
	to look like this:

	```py
	def test_that_number_is_divisible_by_three():
	    assert fizzbuzz.is_divisible_by_three(3) == True
	```

	`assert` is a function given to us by `pytest`. It means that we expect the execution
	of `fizzbuzz.is_divisible_by_three(3)` to return `True`.

1. Run the test: `pytest`.
	This should fail with something like `NameError: name 'fizzbuzz' is not defined`.
	Which makes sense; we haven't created 'fizzbuzz' yet. Time to write some code.

[How your project should look](https://github.com/Callisto13/fb-py/tree/e26fce233624b20152b85761b4a85b2b56f71666) at this stage.

## Part 3: Make it green
Now that we have our first failing test we are going to follow the errors given
by `pytest` to make it pass.
So let's start with the first error we were given: `NameError: name 'fizzbuzz' is not defined`

**Steps**:
1. It's a little vague, but it is essentially saying that it doesn't know of any package called `fizzbuzz`.
	In python you need to specify the packages you want to use at the top of the file, so let's add this line
	at the top of our `fizzbuzz_test.py`:

	```py
	import fizzbuzz
	```
1. When we run our test again, we get a new error: `ModuleNotFoundError: No module named 'fizzbuzz'`.
	Fair enough. We are trying to import a module which does not exist.

1. Making a new module is easy: create a new file called `fizzbuzz.py`.

1. Run the test again, and the error has changed again: `AttributeError: module 'fizzbuzz' has no attribute 'is_divisible_by_three'`.
	'Attribute' can mean many things like a variable or a function. In this case it is going to be a function, so
	let's create that.

1. Open `fizzbuzz.py` and define the function. Don't make it do anything, just define and return:

	```py
	def is_divisible_by_three(number):
	    return
	```

1. Save the file and run the test again: `assert None == True`.
	It's a little hard to read, I'll grant you. The test is evaluating our fizzbuzz function, and the result is None.
	Our test is asserting that the result should be true but of course we are returning nothing.
	So let's go give it what it wants.

1. In `fizzbuzz.py`, make `is_divisible_by_three` return `True`:

	```py
	def is_divisible_by_three(number):
	    return True
	```

1. Run the test again. And we're green! Congrats, you have passed your first test.
	Let's go wreck it.

[How your project should look](https://github.com/Callisto13/fb-py/tree/87137c3c8fae8cbdfb8328d02efa2604ceadadc2) at this stage.

## Part 4: Make it red
Obviously we are not done yet. Hardcoding `True` like that is seen as a Very
Bad Thing, and also not very useful for our game. So let's write another test
to force ourselves to do the right thing.

**Steps**:
1. In `fizzbuzz_test.py` define a second `test_` function under the first.

	```py
	def test_that_number_is_NOT_divisible_by_three():
	    assert fizzbuzz.is_divisible_by_three(1) == False
	```

1. Run the tests again. Back to red? 1 failed, 1 passed? `E       assert True == False`?
	Excellent. Time for some maths.

1. Back in `fizzbuzz.py` we need to make our function work out whether the
	`number` it is being passed as an argument (which right now we are ignoring) is
	actually divisible by three. To do that we need to use modular arithmetic: if
	`number` can be evenly divided by 3, it should return 0 (i.e. not have a remainder).

	```py
	def is_divisible_by_three(number):
	    if number % 3 == 0:
	        return True
	    else:
	        return False
	```
1. Now when we run the tests, both should pass. Our function is a bit clunky, so we can slim it down a little.
	The evaluation of `x % y` will return `True` if it equals 0 and `False` if not, so we don't need to explicitly
	check and return:

	```py
	def is_divisible_by_three(number):
	    return number % 3 == 0
	```

1. Seeing as we have very little code
	right now, there is no more refactoring to be done, so we can go on to writing more tests.

[How your project should look](https://github.com/Callisto13/fb-py/tree/d9606ff9d85a54682d455f49a87519412190503c) at this stage.

## Part 5: Around we go again...

**Steps**:
1. In `fizzbuzz_test.py`, add another `test_` function
	to test whether numbers are divisible by 5:

	```py
	def test_that_number_is_divisible_by_five():
	    assert fizzbuzz.is_divisible_by_five(5) == True
	```
1. Run the tests. Do you see `AttributeError: module 'fizzbuzz' has no attribute 'is_divisible_by_five'`?

1. Go define `is_divisible_by_five` in `fizzbuzz.py`. (just define and return, don't make it
do anything.)

1. Run the tests. `E       assert None == True`? Make your function return `True`.

1. Run the tests. Green again?	Go back to your test file and write the opposing
  `test_that_number_is_NOT_divisible_by_five` function.

1. Run the tests. `E       assert True == False`? Fix your code to make it pass.

2 functions in and we are starting to see a pattern here, but let's leave off
refactoring just a little while longer and move onto the last calculation.

By now you should know the routine, so go ahead and write 2 more tests for a function
which knows if a number `is_divisible_by_three_and_five`.

Once you have all 6 tests passing, commit your work and push to github:

```sh
git add fizzbuzz.py fizzbuzz_test.py
git commit -m "knows if numbers are divisible by 3, 5 or 3 and 5"
git push
```

[How your project should look](https://github.com/Callisto13/fb-py/tree/d53f34d98a2a94d581524af4aaf980a6aef7532a) at this stage.

## Part 6: First Refactor
Right now we have three functions which are doing more or less the same thing.
Let's see if we can [DRY](http://programmer.97things.oreilly.com/wiki/index.php/Don't_Repeat_Yourself) this out a bit.
1. In `fizzbuzz.py`, edit your code so that the 3 `is_divisible_by_*` functions
		are replaced by just 1.

	```py
	def is_divisible_by(number, divisor):
	    return number % divisor == 0
	```

1. Run your tests. Are they very unhappy? Update them to use the new function.
  If you are having trouble making them pass, remember to read the failure messages
  carefully: `pytest` is occasionally helpful and will generally point you in the right direction.

[How your project should look](https://github.com/Callisto13/fb-py/tree/574693e0bcfe04b328c6ad41ff095dd3a318fc0c) at this stage.

## Part 7: FizzBuzz says
So now our code can tell us whether a number is divisible by 3, 5 or 3 and 5, but we
can't really play the game with this.

**Steps**:
1. In `fizzbuzz_test.py` define a new `test_` function:

	```py
	def test_says_fizz():
		assert fizzbuzz.says(3) == 'fizz'
	```
	Now we can use `assert` to check whether `fizzbuzz.says(3)` returns 'fizz'.
1. Run your tests. They should fail in a way which by now should be familiar.
1. Go into `fizzbuzz.py` and create that function. (just define. don't implement.)
1. Run the tests again and follow the error message, remember to do just enough
to make them pass.

	```py
	def says(number):
	    return 'fizz'
	```
1. Add the next test to force yourself to write code which actually evaluates something.

	```py
	def test_says_buzz():
		assert fizzbuzz.says(5) == 'buzz'
	```
1. See the test fail, then go back to your code and make your new function process
the argument it is passed by using our `is_divisible_by` function:

	```py
	def says(number):
	    if is_divisible_by(number, 3):
	        return 'fizz'
	    if is_divisible_by(number, 5):
	        return 'buzz'
	```

1. Go back to `fizzbuzz_test.py`, and add the test which check that the
  program says "fizzbuzz" when a number is divisible by 3 and 5.
1. Run the tests, watch it fail.
1. Go to your code and make it pass.

	_note: remember to watch your ordering here. Since you are returning immediately_
	_when the number is first sucessfully divisible, you may end up saying "fizz" rather_
	_than "fizzbuzz". Make sure you check if a number can be divisible by 3 and 5 first._
	_Switch the order of your code to see your tests failing in this way._

1. The last thing we need to do is write a test (and then the code) for when the
  number is not divisible by 3 or 5. It should just return the number.

Once you have all 10 tests passing, commit your work and push to github:

```sh
git add fizzbuzz.py fizzbuzz_test.py
git commit -m "says fizz, buzz and fizzbuzz"
git push
```

[How your project should look](https://github.com/Callisto13/fb-py/tree/9dce7199a3f1847fe4330fa5f4f95d0ccec4d09d) at this stage.

## Part 8: Bonus Round

All our calculations are done and we are ready to play the game... but it doesn't
quite work in the way we planned at the start. We can't do `$ python fizzbuzz.py 3`
and expect to see `fizz` in the terminal right now.

So, for bonus points, you are going to write some integration tests. Up to now we have
been testing out each individual `unit` of code in... unit tests (obvs). Now we
need to verify that our code integrates with the tools which are not directly part of that code
but are going to interact with it (in this case the command line).

You would normally write your `integration` tests in a separate place (the code too, but for this you can ignore that),
so start by creating an `integration_test.py` file and setting up the file in the same
way as we did last time.

In `fizzbuzz.py` you will need to find a way to get the command line arguments
and pass them to your `says` function.

You will also need to find a way of executing your game from inside a test file.
Some examples can be found [here](https://helloacm.com/execute-external-programs-the-python-ways/)
but there are others so take some time to look around.

Drive out your code the same way as you did above: describe the first simple thing
you expect to happen, do just enough to make it pass, then move onto the next simple thing.
Remember to follow the errors returned by `pytest`; 8 times out of 10 the answers will
be there.

To make our game work properly, we want to make sure that:
1. Our code can be executed (successfully).
1. The result will be printed to terminal output (`stdout`).
1. We can pass multiple arguments and see them all processed (`python fizzbuzz 1 2 3` should print `1 2 fizz`).

Once you have all tests passing, commit your work and push to github:

```sh
git add fizzbuzz.py fizzbuzz_test.py integration_test.py
git commit -m "can be run from the command line"
git push
```

[Here](https://github.com/Callisto13/fb-py/tree/d98097b0084153608f7e048dfe6a78d08b2685de) is one way your project could look.

## WooHoo!!

And that's it! You just test-drove your first program.

But don't stop there; test-driven development is a good habit to get into and the
majority of (sensible) companies value it very highly. Think back to small programs
you have written and see if you can do them again through TDD. Or test drive out
another simple game or calculator ([Leap Year](https://github.com/fouralarmfire/square-one/blob/master/tutorials/leap_year.md), maybe?).

There is a testing framework (often much more than one) for every language,
so go ahead and play Fizzbuzz again in the one of your choice.

In fact, small exercises like Fizzbuzz are a great way to get to grips with a new language
and its testing framework; it's my personal goto for the first thing I write in
whatever new thing I am trying.

&nbsp;

**Ready for more TDD in Python? Check out my next tutorial on how to [test-drive a CRUD API](/resources/tutorials/crud-py)!**

&nbsp;

<sup>_Mistakes? Corrections? Suggestions?_ <a href="https://github.com/cbctl/cbctl.github.io/tree/master/tutorials/fizzbuzz-py.md"><svg class="svg-icon"><use xlink:href="/assets/minima-social-icons.svg#github"></use></svg></a></sup>

<sup>_Is something unclear? Do you need help?_ <a href="https://github.com/cbctl/cbctl.github.io/issues/new"><svg class="svg-icon"><use xlink:href="/assets/minima-social-icons.svg#github"></use></svg></a></sup>
