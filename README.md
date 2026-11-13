English version: [README.en.md](README.en.md)

# adherencia-reglas

[![ci](https://github.com/jleonceo/skill-adherencia-reglas/actions/workflows/ci.yml/badge.svg)](https://github.com/jleonceo/skill-adherencia-reglas/actions/workflows/ci.yml)

> Este repositorio es el paquete que se instala como plugin de Claude Code. El método y las cifras están en [adherencia-reglas](https://github.com/jleonceo/adherencia-reglas).

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

## Qué hace

Cuenta ocasiones y respuestas.

Una regla tuya tiene un disparador (escribir un documento, cerrar una jornada, tocar una skill) y
una acción que debería seguirle. La skill recorre el historial de sesiones que Claude Code ya
guarda, localiza los disparadores, mira si la acción llegó después y devuelve la fracción. Eso es
todo. No juzga si la regla es buena. No juzga si el trabajo estuvo bien hecho. No lee nada que no
esté ya en tu disco. Un 20 % puede querer decir que la regla se ignora o puede querer decir que la
regla está mal escrita: esa lectura es tuya.

## Instalación

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

## Uso

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

## Tus reglas

`reglas.json` trae unas de ejemplo para que la primera ejecución devuelva algo en vez de un bloque
de ceros. Después se edita: cada regla lleva un disparador, una acción esperada y, opcionalmente,
un ámbito que la acota a ciertas carpetas.

El ámbito importa más de lo que parece. Una regla que solo aplica a textos publicados, medida sobre
todo lo que escribes, da un porcentaje bajísimo que no significa nada, porque el denominador está
lleno de ocasiones en las que la regla nunca tocó aplicarse. Esto no es una advertencia teórica: la
primera versión de este paquete publicaba en su propia portada una cifra medida así, dentro del
instrumento que existe justamente para cazar ese error, y la encontró un lector de fuera después de
diez revisiones propias que no la vieron.

## Qué no hace

No escribe. Ni en tu historial, ni en tu configuración, ni fuera de la carpeta desde la que se
ejecuta. `test_seguridad.py` planta datos falsos de aspecto sensible (nombres, cuentas, claves) en
un historial de juguete y exige que no salgan por ninguna de las vistas de agregados ni por ningún
camino de error, además de comprobar que no se escribe nada en ningún sitio.

La excepción está declarada a propósito: `--acciones` enseña rutas de tu historial, porque para eso
existe.

## Desinstalar

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

## Por qué fiarse

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

## Requisitos

Python 3.9 o superior, biblioteca estándar, sin red y sin nada que instalar. En macOS y en casi
todo Linux el intérprete se llama `python3`, no `python`: los comandos de esta página van con
`python` porque se escribieron en Windows. Ese 3.9 está **certificado** desde el 27/07/2026: nueve
trabajos en verde, con 3.9, 3.11 y 3.13 sobre Windows, Linux y Mac. Hasta esa mañana aquí ponía
«declarado, no certificado». Era cierto al escribirlo y dejó de serlo con el primer push, pero
siguió publicado unas horas. Una nota de humildad caduca igual que una cifra. Esta caducó hacia el
lado que hace parecer el paquete peor de lo que es.

---

## Licencia

MIT. Ver [LICENSE](LICENSE).
