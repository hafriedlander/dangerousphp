# Dangerous PHP

▸▸▸▸▸▸▸▸

# Not _stupid_ PHP

```php
// This is stupid, and you deserve what you get
eval($_GET['php']);

// Also stupid
`$_GET['shell']`;
```

▸▸▸▸▸▸▸▸
<!-- .element: data-background="images/titanic.jpg"-->

# Mostly harmless

▸▸▸▸▸▸▸▸

# Ambiguity where?

▾▾▾▾

```php
class Foo {
	public $a = 1;
	public function a() {
		return $this->a + 1;
	}
}

$fooInstance = new Foo();
echo $fooInstance->a . "\n";
echo $fooInstance->a() . "\n";
```

▸▸▸▸▸▸▸▸

# Don't have a breakdown

▾▾▾▾

```php
function everyoneKnowsBreak() {
	while(1==1) {
		break;
	}
}

everyoneKnowsBreak();
```

▾▾▾▾

```php
function youCanBreakSeveralLevels() {
	while(1==1) {
		while (1==1) {
			break 2;
		}
	}

	echo "No infinite loop\n";
}

youCanBreakSeveralLevels();
```

▸▸▸▸▸▸▸▸

# Super indirect

▾▾▾▾

```php
$a = 1;
$b = 'a';

echo "Direct access " . $a . "\n";
echo "Indirect access " . $$b . "\n";
```

▾▾▾▾

```php
$a = 1;
$b = 'a';
$c = 'b';
$d = 'c';

echo "Super indirect access " . $$$c . "\n";
echo "Dollar dollar bill ya'll " . $$$$d . "\n";
```

▸▸▸▸▸▸▸▸
<!-- .element: data-background="images/wolf.jpg"-->

# Mildly dangerous

▸▸▸▸▸▸▸▸

# Considered harmful

▾▾▾▾

```php
function yesGotoWorksInPHP() {
	$a = 0;

	whoneedsloops:
	if ($a < 5) {
		$a += 1;
		goto whoneedsloops;
	}

	echo $a . "\n";;
}

yesGotoWorksInPHP();
```

▾▾▾▾

```php
function evenFromExceptionHandlers() {
	$a = 0;
	try {
		hurgh:
		if ($a < 5) {
			$a += 1;
			throw new Exception();
		}
	}
	catch (Exception $e) {
		goto hurgh;
	}

	echo $a . "\n";
}

evenFromExceptionHandlers();
```

▾▾▾▾

```php
function orIntoExceptionHandlers() {
	$e = 1;
	goto whatidonteven;

	try {
		throw new Exception();
	}
	catch (Exception $e) {
		whatidonteven:
		echo $e . "\n";
	}
}

orIntoExceptionHandlers();

```

▸▸▸▸▸▸▸▸

# The static, it does nothing

▾▾▾▾

```php
class Foo {
	function method() {
		echo "method\n";
	}

	static function static_method() {
		echo "static\n";
	}
}

Foo::method();

$fooInstance = new Foo();
$fooInstance->static_method();
```

▸▸▸▸▸▸▸▸

# Neither P in PHP is for Privacy

▾▾▾▾

```php
class Foo {
	private $a = 1;
	private function method() {
		echo "keep out\n";
	}
}

$fooInstance = new Foo();
$reflection = new ReflectionObject($fooInstance);

$prop = $reflection->getProperty('a');
$prop->setAccessible(TRUE);
echo $prop->getValue($fooInstance) . "\n";

$method = $reflection->getMethod('method');
$method->setAccessible(TRUE);
$method->invoke($fooInstance);
```

▸▸▸▸▸▸▸▸

# Undead PHP

▾▾▾▾

```php
function runFirst() {
	echo "first\n";
	nonexistant();
}
function runSecond() {
	echo "second\n";
}

$callList = ['runFirst', 'runSecond'];

function callNext() {
	global $callList;

	if ($next = array_shift($callList)) {
		$next(); callNext();
	}
}
```

▾▾▾▾

```php
ini_set('display_errors', 0);
register_shutdown_function('callNext');

set_exception_handler('callNext');

callNext();
```

▸▸▸▸▸▸▸▸
<!-- .element: data-background="images/aviator.jpg"-->

# Chainsaw-nunchucks dangerous

▸▸▸▸▸▸▸▸

# Ev(a|i)l code generation

▾▾▾▾

```php
function defineBarWithEval() {
	eval("function bar(){ echo 'bar\n'; }");
}

defineBarWithEval();
bar();

function defineFooWithEval() {
	eval("class Foo { function bar() { echo 'method bar\n'; } }");
}

defineFooWithEval();
$fooInstance = new Foo();
$fooInstance->bar();
```

▸▸▸▸▸▸▸▸

# Inclusive code generation

▾▾▾▾

```php
function defineBarWithInclude() {
	file_put_contents(
		__DIR__."/temp.php",
		"<"."?php function bar(){ echo 'bar\n'; }"
	);
	include __DIR__.'/temp.php';
	unlink(__DIR__.'/temp.php');
}

defineBarWithInclude();
bar();
```
▾▾▾▾

```php
function defineFooWithInclude() {
	file_put_contents(
		__DIR__."/temp.php",
		"<"."?php class Foo {
				function bar() { echo 'method bar\n'; }
		}"
	);

	include __DIR__.'/temp.php';
	unlink(__DIR__.'/temp.php');
}

defineFooWithInclude();
$fooInstance = new Foo();
$fooInstance->bar();
```

▸▸▸▸▸▸▸▸

# Auto-includify

▾▾▾▾

```php
spl_autoload_register(function($class){
	eval("
		class $class {
			function bar(){
				echo __CLASS__ . ' bar \n';
			}
		}
	");
}, true, true);

$fooInstance = new Foo();
$fooInstance->bar();
```

▸▸▸▸▸▸▸▸

# Do not spindle, mangle or mutilate this code

▾▾▾▾

```php
spl_autoload_register(function($class){
  $tempFile = __DIR__.'/temp.php';

	$source = file_get_contents(__DIR__ . '/lib/' . $class . '.php');
	file_put_contents($tempFile, str_replace('bar', 'baz', $source));
	include $tempFile;
	unlink($tempFile);

	return true;
}, true, true);

$fooInstance = new Foo();
$fooInstance->baz();
```

▸▸▸▸▸▸▸▸

# Surely this is madness?

▾▾▾▾
<!-- .element: data-background="images/shirley.png"-->

### Don't call me Shirley

- [Error Control Chain](https://github.com/silverstripe/silverstripe-framework/blob/master/Core/Startup/ErrorControlChain.php)
- [bind_manipulation_capture](https://github.com/silverstripe/silverstripe-fulltextsearch/blob/master/code/search/SearchUpdater.php#L20)
- [Phockito](https://github.com/hafriedlander/phockito/blob/master/Phockito.php#L181)


▸▸▸▸▸▸▸▸

# Thank you

<img width=250 src="images/hamish.jpg" /> | <img style="background: white" width=250 src="images/hqlogo.png" />
------ | ------
@hafriedlander | @hotelquickly
