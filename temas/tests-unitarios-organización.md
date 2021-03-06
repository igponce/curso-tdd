# Preparando los tests unitarios

## Planteamiento

Los lenguajes de programación modernos tienen todo un marco de
utilidades y bibliotecas para poder ejecutar esos tests; estos marcos
van desde las funciones que te permiten comprobar si el resultado
obtenido es igual al deseado, hasta las herramientas de construcción o
ejecutores de tareas que se usan, de forma más o menos estándar, en
cada lenguaje de programación para lanzar los tests. En este tema
veremos cómo se organizan los tests, y cómo podemos preparar los
programas que los ejecutan.

## Al final de esta sesión

El estudiante habrá programado algunos tests y habrá hecho ejecuciones
de prueba de forma genérica.

## Criterio de aceptación

La entidad principal del problema se habrá implementado en una o
varias clases, y cada una de las funciones tendrá un test que se ejecutarán en
local. No es necesario que todos los tests pasen, en esta fase.

## Tests unitarios

Aparte de las pruebas de calidad más o menos intrínseca e
independientes del contexto (que hemos visto hasta ahora), es evidente
que eventualmente hará falta asegurarse de que el código cumple las
especificaciones funcionales que se piden. Las pruebas o tests que se
hagan sobre él deben de corresponder a las especificaciones que queremos
que respete nuestro software, y como tales deben de ser previas a la
escritura del código en sí y del test. Es esencial entender
perfectamente qué queremos que nuestro software haga, y hay diferentes
formas de hacerlo, pero generalmente, en metodologías ágiles, tenemos
que hacerlo mediante [*historias de usuario*](https://es.wikipedia.org/wiki/Historias_de_usuario),
narrativas de qué es lo que puede hacer un posible usuario y qué es lo
que el usuario debería esperar; ya hemos hablado de ellas en el tema
dedicado al [diseño](diseño.md). Estas historias de usuario se
convertirán en *issues* del repositorio, agrupados en mojones o hitos, cuyo cierre marcará que el
código está escrito, testeado, y se ajusta a la misma.

En la mayoría de los lenguajes de programación y especialmente en
lenguajes como `node` hay dos niveles en el test: el primero es el
marco de pruebas y el segundo la biblioteca (o bibliotecas) de pruebas
que efectivamente se está usando. El marco de pruebas incluirá
generalmente un script que será el que ejecute todos los tests,
examine el resultado y emita un informe, que dependerá de si los tests
se han superado o no. Un tercer nivel, implícito, es el gestor de
tareas desde el que se lanzará el marco de pruebas que ejecutará los
tests. Pero lo imprescindible, en todos los casos, es que se testeen
las historias de usuario y que los tests sean *siempre* independientes
de la implementación.

> Para ello, todas las bibliotecas de tests emiten sus resultados en
> un formato de texto estándar, que se llama
> [TAP](https://en.wikipedia.org/wiki/Test_Anything_Protocol). Por eso
> los marcos de pruebas se pueden usar con cualquier biblioteca de
> pruebas, incluso de cualquier lenguaje.

Por debajo del marco de pruebas (la biblioteca que permite estructuras
las pruebas), a veces existe una biblioteca externa de aserciones, que son las
diferentes pruebas unitarias que se deben pasar o no. En muchos casos,
la biblioteca de pruebas incluye ya aserciones; en otros casos,
bibliotecas de pruebas pueden trabajar con diferentes bibliotecas de
aserciones; en todo caso, lo que siempre existe es una biblioteca de
aserciones en todos los lenguajes de programación que permiten
comparar el resultado esperado con el resultado obtenido de diferentes
formas, o, en algunos casos, simplemente que se el código se puede
ejecutar correctamente.

Los tests unitarios deben ser, además, unitarios y valga la
redundancia. Dado que cada función debe tener una responsabilidad y
sólo una, esa única responsabilidad debe ser testeada desde el marco
de pruebas. Cada aserción debe
testear sólo una funcionalidad o característica, de forma que si falla
sepamos exactamente cuál ha sido. Además, si el lenguaje de
programación se organiza en subtests o funciones que testean algo,
cada función debe lanzar tests relacionados sólo con una
funcionalidad, si es posible, uno solo. De esa forma, cuando falla un
test sólo se tiene que arreglar lo relacionado con esa funcionalidad,
y no hay que buscar el error dentro de todas las que se están
testeando. El nombre de estas funciones, además, debe ser lo más
explícito posible, dejando totalmente claro lo que testea, si es
posible usando la misma frase que se use en la historia de usuario.

El código debería ser fácil de testear. Si es complicado, es el
momento de refactorizarlo; código complicado de testear también es
complicado de usar.

## Fases de test: *setup*, *tests*, *teardown*

En las fases del proceso de prueba:

* Planificación: en esta fase se decide cuantos tests se van a
  ejecutar, y se informa al marco de tests de esa decisión. Puede ser
  que no se decida de antemano, pero en ese caso habrá que informar
  cuando se han terminado los tests.

* Durante el *setup* se crearán los objetos y cargarán los ficheros
  necesarios hasta poner nuestro objeto en un estado en el que se
  puedan llevar a cabo los tests. Esto puede incluir, por ejemplo,
  también crear esos objetos. En esta fase se pueden crear lo que se
  denominan *fixtures*: son simplemente objetos que, si no funciona su
  creación, algo va mal, pero si funciona se usan como base para el
  resto de los tests. Como se indica en la fase de *teardown* más
  abajo, algunos consideran un *olor de código* usar este tipo de
  estructuras de datos. Así que hay que tomar decisiones en cada caso
  y para cada lenguaje.

* A continuación se llevan a cabo las pruebas en sí; esas pruebas
  pueden estar agrupadas en subtests, y en el caso de que falle un
  subtest fallará el test del que depende completo.

* En la fase de *teardown* se limpia todo lo creado temporalmente, se
  cierran conexiones, y se deja al sistema, en general, en el mismo
  estado en el que estaba al principio.


Diferentes lenguajes tienen diferentes técnicas, más o menos formales,
para llevar a cabo las diferentes fases. Normalmente es parte de la
biblioteca de aserciones decidir si una parte del código se va a
ejecutar o no. Por ejemplo, en los [tests en Perl](t/proyecto.t) que
se pasan en este mismo repositorio, nos interesa ejecutar algunos sólo
cuando se trata de un `pull request`. Usamos esto:

```
# Carga bibliotecas...

unless ( $ENV{'TRAVIS_PULL_REQUEST'} ) {
  plan skip_all => "Check relevant only for PRs";
}
# Resto del programa
```

En Perl se usa la biblioteca de aserciones `Test::More`. Por omisión,
se trabajará *sin plan* y los tests terminarán cuando se ejecute
`done_testing`. Se le puede decir también cuantos tests de van a
ejecutar; todo ello con la orden `plan`. Pero también se puede usar
esta orden para indicarle, como aquí, `skip_all`, que se salte los
tests a menos que (`unless`) Travis nos haya indicado (a través del
valor de una variable de entorno) que se trata de un `pull request`.

En la fase de planificación se deben decidir cuantos tests se van a
ejecutar. Aunque la respuesta obvia podría ser "todos", lo cierto es
que los tests van a variar dependiendo del entorno. Para empezar, hay
que decidir si va a haber algún plan (es decir, si sabemos de antemano
el número de tests) o simplemente se van a ejecutar a continuación
todos los tests que nos encontremos.


Tras la planificación, que es implícita en muchos marcos de tests y
bibliotecas, se ejecutará la fase de setup. Esta fase, en muchos
casos, se tratará
simplemente de las primeras órdenes de un script para organizarlo, y
los últimos para cerrar las pruebas.

Por ejemplo, en Raku estas serían las primeras líneas de
un [test](../ejemplos/raku/t/02-project.t):

```raku
use Test;

use Project::Issue;
use Project::Milestone;
use Project;

constant $project-name = "Foo";
my $issue-id = 1;
my $p = Project.new( :$project-name );
```

En este caso lo único que se hace es crear un objeto, que es sobre el
que van a recaer los tests más adelante. En el caso del módulo
similar, para hitos de una asignatura, en
Go,
[tendremos esto](https://github.com/JJ/HitosIV/blob/master/HitosIV_test.go#L9-L12):

```go
func TestMain(m *testing.M) {
	ReadsFromFile("./hitos_test.json") // Alternative test file
	os.Exit(m.Run())
}
```

`go test` llamará a la función `TestMain` del paquete antes que a
cualquier otra función. El argumento que recibirá no es de tipo
`testing.T`, como en el resto de los casos, sino `testing.M`; esta
función actuará como *setup*, en este caso leyendo de un fichero que
cargará la estructura de datos sobre la que actuarán el resto de los
tests. De hecho, el resto de los tests tenemos que llamarlos
explícitamente (con `m.Run`) y también que salir explícitamente del
`main` usando `os.Exit`, que devolverá el código de salida adecuado.


Lo más importante antes de ponerse a escribir los tests es que *hay
que comprobar comportamientos, no implementación*. Cualquier test que
dependa de la implementación es un impedimento para la
refactorización, y por tanto no es un buen tests. Los comportamientos,
por supuesto, serán los que estén en las historias de usuario, y esto
se debe reflejar también en la "narrativa" del repo: todo el código de
test que se cree debe referenciar a las HUs e issues para los cuales
ha sido creado.


### Usando fixtures en Python

> El ejemplo en Python está en el directorio [`python` dentro de los
> ejemplos](`../ejemplos/python`).

Python, a base de no querer extender la sintaxis, acaba añadiendo
conceptos y construcciones sintácticas para temas inesperados, como
por ejemplo los objetos que se crean en la fase de setup de los tests,
que se denominan *fixtures*, y tienen su sintaxis específica. Para
testear la clase `Issue` que hemos generado anteriormente, se usarían
fixtures de esta forma:

```Python
import pytest
from Project.Issue import Issue, IssueState

PROJECTNAME = "testProject"
ISSUEID = 1

@pytest.fixture
def issue():
    issue = Issue(PROJECTNAME,ISSUEID)
    return issue

def test_has_name_when_created(issue):
    assert issue.projectName  == PROJECTNAME

def test_has_id_when_created(issue):
    assert issue.issueId  == ISSUEID

def test_is_open_when_created(issue):
    assert  issue.state == IssueState.Open

def test_is_closed_when_you_close_it(issue):
    issue.close()
    assert  issue.state == IssueState.Closed

def test_is_open_when_you_reopen_it(issue):
    issue.reopen()
    assert  issue.state == IssueState.Open

```

`fixture` es una orden de `pytest` que crea  un *decorador*, por eso lleva
la arroba delante. En la práctica, crea un objecto del mismo nombre
que la función que se decora que es el que vamos a usar en el resto de
los tests, tal como se ve aquí. En este caso usamos la librería de
aserciones de `pytest` también, ya puestos; en todo caso, lo que
usamos es una simple aserción que comprobará que la expresión que se
le pasa es cierta.

> Hago notar que, una vez más, Python usa convenciones donde otros
> lenguajes usan sintaxis, y eso, desde mi punto de vista, no es
> bueno. Algo EN MAYÚSCULAS es una constante. Que podemos variar de
> valor cuando queramos, pero eso, que es constante, no la toquéis.

Para fixtures, en Python y en cualquier otro lenguaje, hay que seguir
una [serie de buenas
prácticas](https://salmonmode.github.io/2019/03/29/building-good-tests.html).

- Cada *fixture* debe hacer sólo una cosa.

- Se debe testear el resultado final, no el setup. Se trata siempre de
  testear comportamientos, no implementaciones.


Si seguimos estos principios, algunos de los tests arriba son
innecesarios, por ejemplo los dos primeros. Los dos últimos sí testean
comportamientos, y además siguen el principio de que se testea cuando
se cambia el estado. O sea, bien.

## Teardown: dejándolo todo como estaba

Uno de los principios
[FIRST](https://medium.com/@tasdikrahman/f-i-r-s-t-principles-of-testing-1a497acda8d6)
es la I de *isolated* o aislado; los tests deben de ser capaces de
comportarse perfectamente por sí solos, sin depender del entorno y de
cosas como el orden en el que se hayan ejecutado los tests. Cuando se
termina, se deben encontrar en el mismo estado en el que se empezó. Y
también debería ser posible ejecutar los tests en paralelo, de forma
que se deben de tratar de evitar condiciones de carrera sobre los
ficheros o bases de datos en las que se trabaja. Es buena costumbre
usar, por ejemplo, nombres aleatorios, o simplemente ejecutar los
tests en paralelo y ver si hay algo que pete.

> Una visión alternativa es que tanto setup como teardown [son
> perniciosos](https://stackoverflow.com/q/1087317/891440), y por eso
> hay muchos frameworks que no los tienen, y otros que lo han
> eliminado. Es posible que alternativas de grano más fino, que ejecutan
> cosas antes y después de cada test o subtest, sean mejores opciones.

La mayor parte de las veces que se usa este tipo de código es, como se
afirma [aquí](https://stackoverflow.com/a/1087483/891440), porque se
trata de tests de integración; así que lo veremos con más dedicación
más adelante.

## Actividad

A partir del diseño creado en la anterior actividad, y siguiendo las
prácticas de uso de los issues (y su cierre desde un *commit*), crear
una o varias clases básicas que correspondan a la misma entidad (según
el dominio del problema que se haya elegido), por supuesto incluyendo
los tests correspondientes. Los tests se ejecutarán en local, por lo
pronto. Pasamos ya a la versión `9.x.x` del proyecto.

Se editará el fichero `agil.yaml` añadiéndole, además, la siguiente
clave (sin borrar las anteriores):

```yaml
test:
    - tests/fichero-test.jl
aserciones: "@test"
```

En vez de este nombre ficticio, se usará el nombre del fichero de
construcción que se haya usado para ejecutar los tests, que tendrá que
estar presente en el repositorio.

