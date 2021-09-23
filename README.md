

# JS Intro-övningar

För att kunna köra JS utanför webbläsaren använder vi [Node.js](https://nodejs.org/en/) Node.js (eller Node) är en JS-körtidsmiljö baserad på V8-körtidsmiljön från Chrome.

## Installation

Installera Node med `brew update && brew install node`

Med node följer en [REPL (Read-Eval-Print-Lopp)](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) - liknande Rubys `irb` eller Elixirs `iex`.

Starta repln genom att skriva `node` i terminalen.

```js
➜  ~ node
Welcome to Node.js v16.9.1.
Type ".help" for more information.
> 1 + 2
3
> "hello, world".toUpperCase()
'HELLO, WORLD'
```

För att stänga ner din repl, kan man antingen trycka `ctrl+c` 2 gånger eller skriva `.exit`

Repls är bra när man vill testa kodsnuttar eller försöka klura ut hur t.ex en funktion fungerar, utan att behövera skapa, spara och köra filer.

## Syntax och Evolution

Det sägs att Brendan Eich skapade Javascript på 10 dagar, och syftet var att göra enkla script till webbsidor, inte avancerade applikationer (som det används till nu). Tidsbegränsningen och avgränsningen har lett till en del egenheter i språket.

## Variabler och Scope

Från början skapade man variabler i javascript på följande sätt:

````js
x = 1
````

Problemet med att skapa variabler på detta sätt i js är att variabelns [scope](https://en.wikipedia.org/wiki/Scope_(computer_science)) blir globalt. Globala variabler vill man av varje skäl undvika då det blir mer eller mindre omöjligt att förstå vad som finns i en variabel.

Se nedanstående exempel:

````js
function foo() {
   y = 3;
   return "foo" + y;
}
x = foo()
Console.log(x) //=> 'foo3'
Console.log(y) //=> 3
````

Fast variablen `y` är definerad inne i funktionen `foo()` går den att komma åt utfanför funktionen, dess scope blir globalt.

I js finns det totalt 4 olika nivåer av scopes:

1. Global - Synligt överallt (som i exemplet ovan)
2. Function - Synlig inom en funktion (och dess sub-block/scopes)
3. Block - Synlig inom ett block (och dess sub-block/scopes)
4. Module - Synlig inom en modul (en samling funktioner)

Förenklat kan man säga att varje gång du skriver `{` i js öppnas ett nytt scope (som sen stängs med `}`)

```js
// Utanför funktioner: Global scope
const first_name = "Test";
const last_name = "Testson";
const test_results = [18, 12, 16, 23, 16];

function full_name() { // nytt scope (funktion)
  return `${first_name} ${last_name}`;
}

function max_test_result() { // nytt scope (funktion)
  let i = 0;
  let max = 0;
  while (i < test_results.length) {// nytt scope (while)
    const current_test_result = test_results[i];
    if (current_test_result > max) { // nytt scope (if)
      max = current_test_result;
    }
    i += 1;
  }
  return max;
}
```

Följande sätt finns för att deklarera variabler i js:

### Var

Variabler deklarerade med nyckelordet **`var`** har **function scope**, dvs de går att använda överallt inne i funktionen de är deklarerade (om de inte är deklarerade utanför en funktion, då får de global scope.)

````js
var foo = 1 // foo är global

function bar() {
  var baz = 2; // baz går att använda överallt inne i funktionen
  if (foo == baz) { // foo är ju global
    var qux = 3; //qux går att använda överallt inne i funktionen
  }
  return qux; // qux är deklarerad inne i if-blocket, men går att komma åt utanför 
}
````

Vad händer om if-satsen aldrig körs? Jo, qux blir `undefined`. Antagligen inte något man vill.

### Funktionsparametrar

Variabler som kommer via en funktionsdefinition får samma scope som variabler deklarerade med `var`

````js
function foo(bar, baz) {
  return bar + baz;
}

foo(1, 2) // 3
Console.log(bar) // undefined (bar finns enbart inne i funktionen foo)
Console.log(baz) // undefined (baz finns enbart inne i funktionen foo)
````

### Let

Variabler som är deklarerade med nyckelordet  **`let`** har **block scope**, det vill säga att de enbart går att använda i det scope de är deklarerade i (och i eventuella nästade scopes.)

```js
function foo(bar, baz) {
  if (bar < 18) {
    let qux = 10; //qux finns tillgängligt inne i if-satsen, men inte utanför
    if baz >= 65 {
      qux -= 5; //qux finns tillgängligt i den nästade if-satsen (nästat scope)
    }
  }
  return qux; // fel - qux finns inte tillgänglig här
}
foo(17,76); //Uncaught ReferenceError: qux is not defined
```

### Const

Variabler som är deklarerade med nyckelordet **`const`** har precis som variable deklarerade med **`let`** **block scope**, det vill säga att de enbart går att använda i det scope de är deklarerade i (och i eventuella nästade scopes.), med följande skillnad: **`const`** skapar en read-only variabel, det vill säga man får inte tilldela ett annat värde till variabeln, den är så att säga, *konstant*. 

```js
function foo(bar) {
  const baz = 10; //qux får inte tilldelas ett nytt värde
  if (bar < 18) {
      baz -= 5; //försök till modifiering av qux
    }
  return qux;
}
foo(17); //Uncaught TypeError: Assignment to constant variable.
```
Observera dock att om variabeln refererar till t.ex ett objekt, map eller array får objektets/arrayens/mapens interna state förändras:
```js
function qux(zoop) {
  const thud = [10]; //thud tilldelas en array
  if (zoop < 18) {
      thud.push(5); //ett nytt värde läggs på arrayen (tillåtet, det är "samma" array)
    }
  return thud;
}
qux(17); // [10, 5]
```

### Initiering av variabler utan värde

Variabler deklareade med **`let`** går att deklarera utan att tilldela ett värde (de tilldelas automagiskt värdet `undefined`).

Variabler deklarerade med **`const`** måste tilldelas ett värde när de deklareras

```js
function foo(bar) {
    const baz = 10; // const; tillåtet, ett värde tilldelas vid deklararationen
    //const qux; //const; inte tillåtet, inget värde tilldelas (Uncaught SyntaxError: Missing initializer in const declaration)
    let zoop, qirp; //let; tillåtet, variabeln zoop och qirp deklareras utan värde (initieras automagiskt till `undefined`)
    let tink = 5; //let; tillåtet
    if (bar < 18) {
        zoop = 10;
    }
    return [baz, tink, zoop, qirp]; //tillåtet, men qirp kommer vara `undefined`
  }
const grulp = foo(17);  // [ 10, 5, 10, undefined ]
```

När du försöker göra något med värden som är `undefined` kommer dit program få problem:

### Rekommendation

 -1. Använd inte globala variabler

0. Använd alltid **`let`** eller **`const`** när du deklarerar variabler.

1. Föredra  **`const`** framför **`let`** (och ändra till **`let`** om det visar sig att det inte går).

2. Deklarering

   a. Deklarera alla variabler i början av funktionen

   b. Skapa inte nya variabler inne i t.ex if-satser eller loopar (om du inte är helt säker på att de inte kommer användas någon annan stans).

   c. Tilldela alla variabler ett explicit värde (låt dem inte bli `undefined`)

   ```js
   function foo(bar) {
     const baz = 10
     let qux = 0
     let zoop = bar + 4
     ... //resten av funktionen (inga nya variabler introduceras)
   }
   ```

## Jämförelser

Js är svagt typat, det vill säga, det operationer som i de flesta tillåtna språk inte hade varit tillåtna *kan* vara tillåtna i js, t.ex addera ett tal och en sträng

```js
1 + '3' // '13'
'3' + 1 // '31'
```

Se [Wat](https://www.destroyallsoftware.com/talks/wat).

Detta är problematiskt, men extra problematiskt vid jämförelser:

```js
'3' == 3; // true
```

Jämförelseoperatorn `==` gör typomvandlingar för att se till att båda operanderna har samma datatyp. 

Det finns även jämförelseoperatorn `===` ("triple equals") som inte gör typomvandlingar:

```js
'3' === 3; // false
```

### Rekommendation

0. Använd **alltid** `===` vid jämförelser

