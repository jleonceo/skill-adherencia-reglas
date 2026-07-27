# adherencia-reglas

**Escribes reglas para tu agente de IA y no sabes si las cumple. Esta skill lo cuenta sobre el
historial que Claude Code ya guarda en tu disco, y te devuelve un porcentaje por regla.**

![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue)
![sin dependencias](https://img.shields.io/badge/dependencias-ninguna-brightgreen)
![155 casos](https://img.shields.io/badge/banco-155%20casos-brightgreen)
![mutación 15/15](https://img.shields.io/badge/mutaci%C3%B3n-15%2F15-brightgreen)
![licencia MIT](https://img.shields.io/badge/licencia-MIT-lightgrey)

```
/plugin marketplace add jleonceo/adherencia-reglas-skill
/plugin install adherencia-reglas@adherencia-reglas-skill
```

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
/plugin marketplace add jleonceo/adherencia-reglas-skill
/plugin install adherencia-reglas@adherencia-reglas-skill

# 2. Copiando la carpeta a tus skills personales
git clone https://github.com/jleonceo/adherencia-reglas-skill
cp -r adherencia-reglas-skill/skills/adherencia-reglas ~/.claude/skills/

# 3. Sin instalar nada, ejecutando el script directamente
python skills/adherencia-reglas/medir_adherencia.py
```

La tercera vía existe porque el instrumento no necesita a Claude Code para funcionar: es un script
de biblioteca estándar que lee ficheros `.jsonl`, y si lo único que quieres es el número, te lo da
sin tocar tu configuración ni dejar rastro en ella.

### Uso

```bash
python medir_adherencia.py --sesiones "~/.claude/projects/MI-PROYECTO"
```

Cambia `MI-PROYECTO` por el nombre de tu carpeta. Mira qué hay dentro de `~/.claude/projects` y usa
la del proyecto que quieras medir. Deja las comillas puestas: ese nombre lo fabrica Claude Code a
partir de tu ruta de trabajo y puede traer espacios dentro. Sin argumentos mide todos a la vez.

| Opción | Para qué |
|---|---|
| `--sesiones RUTA` | qué historial se mide, un proyecto o la raíz que los contiene |
| `--reglas FICHERO` | tus reglas, si no quieres las de ejemplo |
| `--por-dia` | la tasa día a día, para ver si sube o baja |
| `--acciones` | qué acciones reconoció y en qué ficheros, para calibrar tu vocabulario |
| `--json` | la salida en JSON, para encadenarla con otra cosa |

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
# si la instalaste como plugin
/plugin uninstall adherencia-reglas@adherencia-reglas-skill

# si copiaste la carpeta
rm -rf ~/.claude/skills/adherencia-reglas
```

No deja nada detrás: ni caché, ni base de datos, ni fichero de estado.

### Por qué fiarse

**155 casos en tres bancos**, todos sobre trazas fabricadas y nunca sobre el historial real, porque
un banco que dependa de los datos de hoy cambia de resultado mañana.

Y los tres están verificados por mutación, que es lo que separa un banco de un adorno:

```bash
python mutar.py
```

Ese comando sabotea el código a propósito, **15 sabotajes**, y cuenta cuántos caza el banco. Hoy
caza los 15 sin un hueco. No hay que creérselo, se ejecuta y se ve. El fichero medido se restaura
al acabar y también si matan el proceso a mitad, porque la copia intacta vive en disco y no en la
memoria del programa; mientras esa copia siga ahí, `test_seguridad.py` se pone rojo.

Parte de esos sabotajes los escribió alguien que no había escrito el código, y son justo los que
los propios no veían. Un mutador escrito por el autor hereda el punto ciego del autor.

### Requisitos

Python 3.9 o superior, biblioteca estándar, sin red y sin nada que instalar. Ese 3.9 está
declarado, no certificado: lo único ejecutado hasta hoy es Python 3.14 y 3.13 en Windows, y quien
lo certifica es la matriz de CI de este repositorio a partir de su primer verde.

---

## English

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
/plugin marketplace add jleonceo/adherencia-reglas-skill
/plugin install adherencia-reglas@adherencia-reglas-skill

# 2. By copying the folder into your personal skills
git clone https://github.com/jleonceo/adherencia-reglas-skill
cp -r adherencia-reglas-skill/skills/adherencia-reglas ~/.claude/skills/

# 3. Without installing anything, running the script directly
python skills/adherencia-reglas/medir_adherencia.py
```

The third way exists because the instrument does not need Claude Code to work: it is a standard
library script that reads `.jsonl` files, and if all you want is the number, it hands it over
without touching your configuration or leaving a trace in it.

### Usage

```bash
python medir_adherencia.py --sesiones "~/.claude/projects/MY-PROJECT"
```

Replace `MY-PROJECT` with your folder name. Look inside `~/.claude/projects` and pick the project
you want to measure. Keep the quotes: Claude Code builds that name from your working path and it
can contain spaces. With no arguments it measures every project at once.

| Option | What for |
|---|---|
| `--sesiones PATH` | which history to measure, one project or the root holding them |
| `--reglas FILE` | your rules, if you do not want the example ones |
| `--por-dia` | the rate day by day, to see whether it climbs or falls |
| `--acciones` | which actions it recognised and in which files, to calibrate your vocabulary |
| `--json` | JSON output, to chain it into something else |

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
# if you installed it as a plugin
/plugin uninstall adherencia-reglas@adherencia-reglas-skill

# if you copied the folder
rm -rf ~/.claude/skills/adherencia-reglas
```

It leaves nothing behind: no cache, no database, no state file.

### Why trust it

**155 cases across three benches**, all on fabricated traces and never on real history, because a
bench that depends on today's data returns something else tomorrow.

And all three are mutation-verified, which is what separates a test bank from an ornament:

```bash
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

### Requirements

Python 3.9 or newer, standard library, no network and nothing to install. That 3.9 is declared, not
certified: the only thing run so far is Python 3.14 and 3.13 on Windows, and what certifies it is
this repository's CI matrix from its first green build onward.

---

## Licencia / License

MIT. Ver [LICENSE](LICENSE).
