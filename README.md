# adherencia-reglas

**Escribes reglas para tu agente de IA y no sabes si las cumple. Esta skill lo cuenta sobre el
historial que Claude Code ya guarda en tu disco, y te devuelve un porcentaje por regla.**

![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue)
![sin dependencias](https://img.shields.io/badge/dependencias-ninguna-brightgreen)
![160 casos](https://img.shields.io/badge/banco-160%20casos-brightgreen)
![mutación 15/15](https://img.shields.io/badge/mutaci%C3%B3n-15%2F15-brightgreen)
![licencia MIT](https://img.shields.io/badge/licencia-MIT-lightgrey)

```
/plugin marketplace add jleonceo/skill-adherencia-reglas
/plugin install adherencia-reglas@skill-adherencia-reglas
```

Esto es lo que devuelve, entero y sin recortar. Sale del ejemplo que viaja dentro del repositorio,
así que se reproduce con `python skills/adherencia-reglas/medir_adherencia.py --sesiones
ejemplo/historial --reglas ejemplo/reglas_ejemplo.json`:

```
  Ampliando accion(es) de fabrica: test. Tu patron va delante, pero el de fabrica sigue
  detras: lo que no case con el tuyo se seguira contando con el suyo.
============================================================================
ADHERENCIA A LAS REGLAS PROPIAS  --  3 sesiones, 31 acciones
============================================================================
  regla                           tocaba  cumplio     tasa  umbral
  ----------------------------------------------------------------------------
  gate-tras-escribir-doc              11        8  72.7 %
      guia del equipo: todo documento pasa el gate de texto antes de darlo por bueno
  suite-tras-tocar-codigo              4        1  25.0 %*
      guia del equipo: si tocas codigo, pasa la suite antes de cerrar

(*) menos de 10 ocasiones: eso no es una tasa, es una anecdota con decimales.

La tasa es de HABITO, no de incumplimiento: una regla puede no aplicar a todos sus
disparadores. Sirve para decidir que hacer con cada una, no para repartir culpas.
```

Ese asterisco es medio paquete. Con cuatro ocasiones no hay tasa que valga, y una herramienta que
te devuelve «25 %» sin decírtelo te está dando una anécdota con decimales disfrazada de medida.

Este repositorio es solo el paquete instalable. La medición sobre historiales reales, con las
cifras y el método, vive aparte: [jleonceo/adherencia-reglas](https://github.com/jleonceo/adherencia-reglas).

[Español](#español) · [English](#english)

---

## Español

### Qué hace

Cuenta ocasiones y respuestas.

Una regla tuya tiene un disparador (escribir un documento, cerrar una jornada, tocar una skill) y
una acción que debería seguirle. La skill recorre el historial de sesiones que Claude Code ya
guarda, localiza los disparadores, mira si la acción llegó después y devuelve la fracción. Eso es
todo. No juzga si la regla es buena. No juzga si el trabajo estuvo bien hecho. No lee nada que no
esté ya en tu disco. Un 20 % puede querer decir que la regla se ignora o puede querer decir que la
regla está mal escrita: esa lectura es tuya.

### Instalación

Tres vías. La primera es la corta y la última funciona siempre.

```bash
# 1. Como plugin de Claude Code
/plugin marketplace add jleonceo/skill-adherencia-reglas
/plugin install adherencia-reglas@skill-adherencia-reglas

# 2. Copiando la carpeta a tus skills personales
git clone https://github.com/jleonceo/skill-adherencia-reglas
cp -r skill-adherencia-reglas/skills/adherencia-reglas ~/.claude/skills/

# 3. Sin instalar nada, ejecutando el script directamente
python skills/adherencia-reglas/medir_adherencia.py
```

El ejemplo de `ejemplo/` se ejecuta sin que tengas historial propio, y es de donde sale el bloque de
salida del principio de esta página:

```bash
python skills/adherencia-reglas/medir_adherencia.py \
  --sesiones ejemplo/historial --reglas ejemplo/reglas_ejemplo.json
```

La tercera vía existe porque el instrumento no necesita a Claude Code. Es un script de biblioteca
estándar que lee ficheros `.jsonl`. Si lo único que quieres es el número, te lo da sin tocar tu
configuración.

### Uso

```bash
cd skills/adherencia-reglas
python medir_adherencia.py --sesiones "~/.claude/projects/MI-PROYECTO"
```

Cambia `MI-PROYECTO` por el nombre de tu carpeta. Mira qué hay dentro de `~/.claude/projects`.

Deja las comillas puestas. Ese nombre lo fabrica Claude Code a partir de tu ruta de trabajo y puede
traer espacios dentro.

Sin argumentos mide todos los proyectos a la vez.

Estas son las diez opciones, que son todas:

| Opción | Para qué |
|---|---|
| `--sesiones RUTA` | qué historial se mide, un proyecto o la raíz que los contiene |
| `--reglas FICHERO` | tus reglas, si no quieres las de ejemplo |
| `--acciones` | qué acciones reconoció y en qué ficheros, para calibrar tu vocabulario |
| `--por-dia N` | desglosa los últimos N días, porque una tasa global esconde el día malo |
| `--curva-ventana` | cómo cambia la tasa según el plazo que le des a la respuesta |
| `--sensibilidad` | cuánto mueve la cifra cada decisión arbitraria, no solo la ventana |
| `--respuesta-en-ambito` | exige que la respuesta caiga en el mismo ámbito que el disparador |
| `--sin-colapsar` | cuenta cada escritura en vez de cada racha de escrituras |
| `--subagentes` | incluye las sesiones de subagentes, que son otro universo |
| `--json` | la salida en JSON, para encadenarla con otra cosa |

`--por-dia` lleva número: `--por-dia 7`. Sin él, el programa para y lo dice.

Las dos que más cambian la lectura son `--curva-ventana` y `--sensibilidad`.

La primera enseña que el plazo que elijas mueve la cifra tanto como el comportamiento que mides. La
segunda enseña cuánto la mueve cada decisión que alguien tomó a ojo al construir el instrumento, que
es información que casi ninguna herramienta de medida publica sobre sí misma.

La primera vez, `--acciones`.

Esa vista es la que evita el malentendido más caro. La skill reconoce acciones por vocabulario. El
tuyo no tiene por qué parecerse al de nadie: si dices «puerta de salida» donde el ejemplo dice
«revisar», tus reglas medirán cero para siempre y ese cero parecerá incumplimiento. `--acciones`
enseña qué encontró y en qué ficheros, así que se ve de un vistazo si el problema está en cómo
describiste la regla o en que de verdad no se cumple.

### Tus reglas

`reglas.json` trae unas de ejemplo para que la primera ejecución devuelva algo en vez de un bloque
de ceros. Después se edita: cada regla lleva un disparador, una acción esperada y, opcionalmente,
un ámbito que la acota a ciertas carpetas.

El ámbito importa más de lo que parece. Una regla que solo aplica a textos publicados, medida sobre
todo lo que escribes, da un porcentaje bajísimo que no significa nada, porque el denominador está
lleno de ocasiones en las que la regla nunca tocó aplicarse. Esto no es una advertencia teórica: la
primera versión de este paquete publicaba en su propia portada una cifra medida así, dentro del
instrumento que existe justamente para cazar ese error, y la encontró un lector de fuera después de
diez revisiones propias que no la vieron.

### Qué no hace

No escribe. Ni en tu historial, ni en tu configuración, ni fuera de la carpeta desde la que se
ejecuta. `test_seguridad.py` planta datos falsos de aspecto sensible (nombres, cuentas, claves) en
un historial de juguete y exige que no salgan por ninguna de las vistas de agregados ni por ningún
camino de error, además de comprobar que no se escribe nada en ningún sitio.

La excepción está declarada a propósito: `--acciones` enseña rutas de tu historial, porque para eso
existe.

### Desinstalar

```bash
# si la instalaste como plugin: son DOS comandos, no uno
/plugin uninstall adherencia-reglas@skill-adherencia-reglas
/plugin marketplace remove skill-adherencia-reglas

# si copiaste la carpeta
rm -rf ~/.claude/skills/adherencia-reglas
```

El segundo comando es el que casi nadie documenta. Desinstalar el plugin lo quita de la lista de
plugins, pero deja el marketplace registrado y su copia local en la caché que mantiene Claude Code
en `~/.claude/plugins/cache`, que es suya y no de esta skill.

De la skill en sí no queda nada, porque no escribe nada: ni base de datos, ni fichero de estado, ni
configuración fuera de su carpeta.

### Por qué fiarse

**160 casos en tres bancos**, todos sobre trazas fabricadas y nunca sobre el historial real, porque
un banco que dependa de los datos de hoy cambia de resultado mañana.

Y los tres están verificados por mutación, que es lo que separa un banco de un adorno:

```bash
cd skills/adherencia-reglas
python mutar.py
```

Ese comando sabotea el código a propósito, **15 sabotajes**, y cuenta cuántos caza el banco. Hoy
caza los 15 sin un hueco. No hay que creérselo, se ejecuta y se ve. El fichero medido se restaura
al acabar y también si matan el proceso a mitad, porque la copia intacta vive en disco y no en la
memoria del programa; mientras esa copia siga ahí, `test_seguridad.py` se pone rojo.

Parte de esos sabotajes los escribió alguien que no había escrito el código, y son justo los que
los propios no veían. Un mutador escrito por el autor hereda el punto ciego del autor.

Y viaja un tercer instrumento, `cobertura.py`, que responde a la pregunta que ningún revisor puede
contestar mirando: qué líneas del programa no ejecuta ni un caso del banco. Usa `trace`, de la
biblioteca estándar, porque aquí no se instala nada.

### Requisitos

Python 3.9 o superior, biblioteca estándar, sin red y sin nada que instalar. En macOS y en casi
todo Linux el intérprete se llama `python3`, no `python`: los comandos de esta página van con
`python` porque se escribieron en Windows. Ese 3.9 está **certificado** desde el 27/07/2026: nueve
trabajos en verde, con 3.9, 3.11 y 3.13 sobre Windows, Linux y Mac. Hasta esa mañana aquí ponía
«declarado, no certificado». Era cierto al escribirlo y dejó de serlo con el primer push, pero
siguió publicado unas horas. Una nota de humildad caduca igual que una cifra. Esta caducó hacia el
lado que hace parecer el paquete peor de lo que es.

---

## English

> **Read this first: the tool speaks Spanish, this page does not change that.** Nine of its ten
> flags, all its configuration keys, the literal values `antes` and `despues`, the fifteen action
> verbs and every line it prints are in Spanish. A rule written with English keys is rejected,
> loudly and in Spanish. This page is the reference for that vocabulary, not a localised program.
>
> The verbs: `git-commit` · `git-add` · `git-push` · `git-mirar` (inspect) · `test` · `shell` ·
> `leer-doc` (read a document) · `leer-codigo` (read code) · `escribir-doc` · `escribir-codigo` ·
> `buscar` (search) · `subagente` · `escribir-doctrina` · `leer-skill` · `leer-rag`. Direction takes
> `antes` or `despues`; the window is `ventana`, the start date `desde`, the scope `ambito` and the
> pass mark `umbral`.


### What it does

It counts occasions and responses.

One of your rules has a trigger (writing a document, closing a working day, touching a skill) and
an action that should follow it. The skill walks the session history Claude Code already keeps,
finds the triggers, checks whether the action came after, and returns the fraction. That is all it
does. It does not judge whether the rule is a good rule. It does not judge whether the work was any
good. It reads nothing that is not already on your disk. A 20 % may mean the rule gets ignored or
it may mean the rule is badly written: that reading is yours.

### Install

Three ways. The first is the short one and the last always works.

```bash
# 1. As a Claude Code plugin
/plugin marketplace add jleonceo/skill-adherencia-reglas
/plugin install adherencia-reglas@skill-adherencia-reglas

# 2. By copying the folder into your personal skills
git clone https://github.com/jleonceo/skill-adherencia-reglas
cp -r skill-adherencia-reglas/skills/adherencia-reglas ~/.claude/skills/

# 3. Without installing anything, running the script directly
python skills/adherencia-reglas/medir_adherencia.py
```

The example that ships in `ejemplo/` runs without any history of your own, which is what the output
block at the top of this page comes from:

```bash
python skills/adherencia-reglas/medir_adherencia.py \
  --sesiones ejemplo/historial --reglas ejemplo/reglas_ejemplo.json
```

The third way exists because the instrument does not need Claude Code. It is a standard library
script that reads `.jsonl` files. If all you want is the number, it hands it over without touching
your configuration.

### Usage

```bash
cd skills/adherencia-reglas
python medir_adherencia.py --sesiones "~/.claude/projects/MY-PROJECT"
```

Replace `MY-PROJECT` with your folder name. Look inside `~/.claude/projects`.

Keep the quotes. Claude Code builds that name from your working path and it can contain spaces.

With no arguments it measures every project at once.

These are the ten options, which is all of them:

| Option | What for |
|---|---|
| `--sesiones PATH` | which history to measure, one project or the root holding them |
| `--reglas FILE` | your rules, if you do not want the example ones |
| `--acciones` | which actions it recognised and in which files, to calibrate your vocabulary |
| `--por-dia N` | breaks down the last N days, because a global rate hides the bad day |
| `--curva-ventana` | how the rate moves with the window you allow the response |
| `--sensibilidad` | how much each arbitrary decision moves the figure, not just the window |
| `--respuesta-en-ambito` | demands the response land in the same scope as the trigger |
| `--sin-colapsar` | counts every write instead of every run of writes |
| `--subagentes` | includes subagent sessions, which are a different universe |
| `--json` | JSON output, to chain it into something else |

`--por-dia` takes a number: `--por-dia 7`. Without it the program stops and says so.

The two that most change the reading are `--curva-ventana` and `--sensibilidad`.

The first shows that the window you pick moves the figure as much as the behaviour you measure. The
second shows how much each decision someone made by eye while building the instrument moves it,
which is information almost no measuring tool publishes about itself.

Start with `--acciones`.

That view heads off the most expensive misunderstanding. The skill recognises actions by
vocabulary. Yours need not resemble anyone else's: if you say "exit gate" where the example says
"review", your rules will measure zero forever and that zero will look like non-compliance.
`--acciones` shows what it found and in which files, so you can tell at a glance whether the
problem is how you described the rule or the rule genuinely going unfollowed.

### Your rules

`reglas.json` ships with example rules so the first run returns something instead of a block of
zeroes. After that you edit it: each rule carries a trigger, an expected action and, optionally, a
scope that narrows it to certain folders.

Scope matters more than it looks. A rule that only applies to published text, measured across
everything you write, yields a dismal percentage that means nothing, because the denominator is
full of occasions where the rule never applied. This is not a theoretical warning: the first
version of this package published exactly such a figure on its own front page, inside the very
instrument built to catch that error, and an outside reader found it after ten in-house reviews
had not.

### What it does not do

It does not write. Not to your history, not to your configuration, not outside the folder it runs
from. `test_seguridad.py` plants fake sensitive-looking data (names, account numbers, keys) in a
toy history and demands that none of it surfaces through the aggregate views or through any error
path, on top of checking that nothing is written anywhere.

The exception is declared on purpose: `--acciones` shows paths from your history, because that is
what it is for.

### Uninstall

```bash
# if you installed it as a plugin: that is TWO commands, not one
/plugin uninstall adherencia-reglas@skill-adherencia-reglas
/plugin marketplace remove skill-adherencia-reglas

# if you copied the folder
rm -rf ~/.claude/skills/adherencia-reglas
```

The second command is the one almost nobody documents. Uninstalling the plugin drops it from the
plugin list, but leaves the marketplace registered and its local copy in the cache Claude Code keeps
at `~/.claude/plugins/cache`, which belongs to Claude Code and not to this skill.

Of the skill itself nothing is left, because it writes nothing: no database, no state file, no
configuration outside its own folder.

### Why trust it

**160 cases across three benches**, all on fabricated traces and never on real history, because a
bench that depends on today's data returns something else tomorrow.

And all three are mutation-verified, which is what separates a test bank from an ornament:

```bash
cd skills/adherencia-reglas
python mutar.py
```

That command sabotages the code on purpose, **15 sabotages**, and counts how many the bank catches.
Today it catches all 15 with no gaps. You do not have to take that on faith, you run it and watch.
The file under test is restored when it finishes and also if the process is killed halfway, because
the intact copy lives on disk and not in the program's memory; while that copy is still there,
`test_seguridad.py` goes red.

Some of those sabotages were written by someone who had not written the code, and they are exactly
the ones the in-house ones could not see. A mutator written by the author inherits the author's
blind spot.

A third instrument travels with the package, `cobertura.py`, which answers the question no reviewer
can answer by reading: which lines of the program no test case ever executes. It uses `trace`, from
the standard library, because nothing gets installed here.

### Requirements

Python 3.9 or newer, standard library, no network and nothing to install. On macOS and on most
Linux the interpreter is called `python3`, not `python`: the commands on this page say `python`
because they were written on Windows. That 3.9 is **certified** as of 27/07/2026: nine green jobs
across 3.9, 3.11 and 3.13 on Windows, Linux and Mac. Until that morning this line read «declared,
not certified». It was true when written and stopped being true on the first push, yet it stayed up
for hours. A note of humility goes stale like any other figure. This one went stale towards the side
that makes the package look worse than it is.

---

## Licencia / License

MIT. Ver [LICENSE](LICENSE).
