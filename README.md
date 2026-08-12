# getopt

Declarative command-line option parsing for MultiValue BASIC — a portable
replacement for hand-parsing a sentence with `FIELD`/`INDEX`.

A verb **declares** its options once, calls `GETOPT.PARSE`, and reads the result
with small accessors. Handles `-x`, `-xVAL`, `-x VAL`, `--long`, `--long=VAL`,
`--long VAL`, the `--` terminator, repeated options, quoted multi-word values,
and a positional remainder.

Pure BASIC. The only per-platform line is `GETOPT.SENTENCE` (`SENTENCE()` on
MVX, `@SENTENCE` on UniData/UniVerse); everything else is the common MV BASIC
core. Tested on MVX and UniData.

## Use

```basic
SPEC = ""
CALL GETOPT.OPT(SPEC, "m", "message", 1, "", "commit message")  ;* -m / --message, takes a value
CALL GETOPT.OPT(SPEC, "o", "open",    0, "", "open format")      ;* -o / --open, a flag

CALL GETOPT.SENTENCE(S)            ;* portable: SENTENCE() or @SENTENCE
CALL GETOPT.PARSE(SPEC, S, 2)      ;* parse, dropping the leading 2 words (verb + subcommand)

IF GO.ERR # "" THEN ...            ;* or: CALL GETOPT.ERR(E)
CALL GETOPT.VAL("m", MSG)          ;* option value ("" if absent; default applied)
CALL GETOPT.HAS("open", ISON)      ;* 1 if the flag/option was present, else 0
CALL GETOPT.ARG(1, FIRST)          ;* Nth positional
CALL GETOPT.NARGS(N)               ;* positional count
```

`GETOPT.PARSE`'s `SKIP` is how many leading words to drop before options begin —
`1` for `VERB ...`, `2` for `VERB SUBCOMMAND ...`. The sentence is a plain
string argument, so it can come from anywhere; `GETOPT.SENTENCE` is optional
sugar. Read the command line **before** any `PERFORM`/`EXECUTE`, which clobber
`@SENTENCE`.

## API

| Subroutine | Purpose |
|---|---|
| `GETOPT.NEW(SPEC)` | reset a spec (`SPEC = ""`) |
| `GETOPT.OPT(SPEC, SHORT, LONG, TAKESVAL, DEFAULT, HELP)` | declare one option (`TAKESVAL=0` ⇒ boolean flag) |
| `GETOPT.PARSE(SPEC, SENTENCE, SKIP)` | parse into `COMMON /GETOPT/` |
| `GETOPT.VAL(NAME, VAL)` | option value by short or long name |
| `GETOPT.HAS(NAME, ON)` | was the option present |
| `GETOPT.ARG(N, VAL)` | Nth positional |
| `GETOPT.NARGS(N)` | positional count |
| `GETOPT.ERR(MSG)` | parse error, if any |
| `GETOPT.USAGE(SPEC, OUT)` | rendered option list for help text |
| `GETOPT.SENTENCE(S)` | portable command-line fetch |

Parsed state lives in `COMMON /GETOPT/ GO.NAMES, GO.VALS, GO.HAS, GO.ARGS,
GO.ERR`; declare it to read `GO.ERR` inline, or use the accessors.

Pairs with the [`cmd`](https://github.com/mvx-lang/mv_cmd) framework: a
subcommand declares its flags with `CMD.FLAG`, and `CMD.RUN` parses them via
getopt before dispatching.

## License

GPL-2.0-only.
