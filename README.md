# Trianguish

Trianguish is a 2d "programming language" designed by Radvylf Programs.

## Basics

Trianguish uses a triangular grid of cells, called "ops" (short for "operator"). Each of these has three inputs or outputs, called "pins", which can be of various types, including inputs and outputs. Ops pass around information via their pins, performing operations such as addition, comparison, and conditonal switching. This occurs in "ticks", the smallest meaningful unit of time in Trianguish, during which all ops simultaneously observe their inputs and compute their outputs. Trianguish features some operators, called builders, which can change the ops around them, and others which can perform I/O operations.

## Data

Trianguish has two data types: NIL and numbers. NILs are used to indicate that no input could be taken, no output could be produced, or that no op or pin corresponds to an input pin. "Vanilla" Trianguish's number type is a three digit base-6 integer, which can represent values from 0 to 215, but this "Mod 216" behavior can be turned on or off, and when off Trianguish will allow numbers to be arbitrary signed integers. As Trianguish is self-modifying, all ops have IDs which are also numbers. There are only 216 meaningful IDs for an op, many of which are redundant (when Mod 216 is off, op IDs can be outside the range of 0 to 215, but they will behave as if their IDs are modulo'd to be within this range). In this documentation, numbers will be represented with trailing `n`s.

## Ops

Ops are stored using numeric IDs, typically ranging from `0n` to `215n`. This is broken into three fields: op type, variant, and orientation. Op type ranges from `0n` to `11n`, variant from `0n` to `2n`, and orientation from `0n` to `5n`. To find these values, the formulas `op_type = floor(id / 18n)`, `variant = floor(id / 6n) % 3n`, and `orientation = id % 6n` may be used.

The op type determines which operation an op will perform:

- `0n`: `nil`
- `1n`: `constant`
- `2n`: `2-wire`
- `3n`: `splitter`
- `4n`: `t-switch`
- `5n`: `s-switch`
- `6n`: `n-switch`
- `7n`: `add`
- `8n`: `mul`
- `9n`: `cmp`
- `10n`: `i/o`
- `11n`: `builder`

The variant field is used only for constants and builders:

- `0n`: `0` or `i-builder`
- `1n`: `1` or `o-builder`
- `2n`: `-1` or `s-builder`

The orientation contains two subfields, rotation and chirality. Rotation is `0n` by default, `1n` for a single pin rotation left, and `2n` for a single pin rotation right ("left" and "right" referring to which side the up/down facing pin rotates toward), and chirality determines what order pins go in, or the "handedness" of an op (you can think of it as drawing the op on a sheet of transparent paper, and flipping the sheet over). Rotation and chirality can be found with the formulas `rotation = floor(orientation / 2n)` and `chirality = orientation % 2n`.

## Op types

**NIL:**

![Nil: All pins gray](docs_imgs/nil.png?raw=true "Nil: All pins gray")

NILs (`0n`) are the most basic op, simply outputting NIL to each of their pins.

**Constant:**

![Constant: All pins blue](docs_imgs/constant.png?raw=true "Constant: All pins blue")

Constants come in three varieties: 0 (`18n`), 1 (`24n`), and -1 (`30n`). Each of these variants will output their value on all pins. If Mod 216 is on, any -1 constants will wrap around to 215.

**2-Wire:**

![2-Wire: Gray, green, and blue pins](docs_imgs/2-wire.png?raw=true "2-Wire: Gray, green, and blue pins")

2-Wires (`36n`) are the most basic way to move values. They have one input pin and one output pin, and will copy their input to their output, with their third pin being NILs.

**Splitter:**

![Splitter: One green and two blue pins](docs_imgs/splitter.png?raw=true "Splitter: One green and two blue pins")

Splitters (`54n`) are similar to 2-wires, but output on both of their non-input pins.

**T-Switch:**

![T-Switch: Purple, green, and blue pins](docs_imgs/t-switch.png?raw=true "T-Switch: Purple, green, and blue pins")

T-Switches (`72n`) act as conditional wires. If non-NIL input is provided to their third pin (differentiated by color), the input and output pins act as a 2-wire, and otherwise the t-switch acts as a NIL.

**S-Switch:**

![S-Switch: Purple, green, and blue pins](docs_imgs/s-switch.png?raw=true "S-Switch: Purple, green, and blue pins")

S-Switches (`90n`) act like two-input wires. They will behave as standard 2-wires, unless their input is NIL, in which case any input to their third pin (differentiated by color) will be outputted. Only if both inputs are NIL will an s-switch output NIL.

**N-Switch:**

![N-Switch: Purple, green, and blue pins](docs_imgs/n-switch.png?raw=true "N-Switch: Purple, green, and blue pins")

N-Switches (`108n`) are used for conditionally switching between NIL and non-NIL outputs based on comparison. If the n-switch's two inputs are different, the primary input will be outputted, and otherwise the n-switch's output will be NIL.

**Add:**

![Add: Two green and one blue pin](docs_imgs/add.png?raw=true "Add: Two green and one blue pin")

Adds (`126n`) add their inputs. Unlike switches, both inputs to an add are identical. If both inputs are non-NIL, the add will output the sum of its inputs (only wrapping if Mod 216 is on), and otherwise it will output NIL.

**Mul:**

![Mul: Two green and one blue pin](docs_imgs/mul.png?raw=true "Add: Two green and one blue pin")

Muls (`144n`) multiply their inputs. Like adds, both inputs to a mul are identical. If both inputs are non-NIL, the mul will output the product of its inputs (only wrapping if Mod 216 is on), and otherwise it will output NIL.

**Cmp:**

![Cmp: Purple, green, and blue pins](docs_imgs/cmp.png?raw=true "Cmp: Purple, green, and blue pins")
    
Cmps (`162n`) compare their two inputs, outputting one of three values to indicate less than, equal to, or greater than. If either input is NIL, the cmp's output will also be NIL. If the primary input to the cmp is less than the secondary input (differentiated by color), the cmp's output is -1 (or 215 if Mod 216 is on), if they are equal it is 0, and if the primary input is greater the output is 1. This is equivalent to taking the sign of the primary input minus the secondary input. Cmps are most useful when chained with n-switches.

**I/o:**

![I/o: Purple, green, and blue pins](docs_imgs/cmp.png?raw=true "I/o: Purple, green, and blue pins")

I/os (`180n`) can perform user input and output operations. Their primary input pin is used for output, with any non-NIL input being outputted, and their secondary input pin is used to take input (so that all input is not consumed before it can be properly handled). If the secondary input pin is given a non-NIL value, one value will be taken from user input. This will be outputted on the i/o's output pin, and will always be non-NIL, unless input is exhausted. Once NIL is returned from an i/o's input, all future input operations will return NIL.

**Builders:**

![I-Builder: Green, brown, and blue pins](docs_imgs/i-builder.png?raw=true "I-Builder: Green, brown, and blue pins")

![O-Builder: Blue, brown, and green pins](docs_imgs/o-builder.png?raw=true "O-Builder: Blue, brown, and green pins")

![S-Builder: Gray, brown, and blue pins](docs_imgs/s-builder.png?raw=true "S-Builder: Gray, brown, and blue pins")
    
Builders come in three varieties: i-builders (`198n`), o-builders (`204n`), and s-builders (`210n`). Builders are one of the most important ops, as they allow for self-modification, in turn permitting complex timing operations, more efficient storage, randomization or undefined behavior, and short non-NIL pulses. Builders are the only ops to have a build input pin, which when given non-NIL input, will treat their input as a numeric representation of an op to "build" in an adjacent cell before the next tick. Builders will always build one pin clockwise of their build input pin, and the pin adjacent to this cell can have varying purposes. An i-builder or o-builder will double as wires, with i- or o- indicating which end is used as the build cell. Possibly the most important builder type is the s-builder, which has a NIL pin adjecent to its build cell, and an output pin. S-Builders will output the numeric representation of the op in their build cell. This will always be non-NIL. If Mod 216 is off, builders can place ops with IDs outside of the typical range (`0n` to `215n`), and s-builders can read these unchanged.

Fun fact: The original idea for Trianguish, years ago, was for it to simulate living organisms, or other complex macro-scale automata. This would not work, since the density of builders to support ops is likely far too low for self-moving structures to be built.

## Timing

During a tick, all ops will update simultaneously. All ops will observe the states of pins around them before any of that tick's changes take place, ensuring that programs work identically when rotated or moved. This has two exceptions, however. First, building will occur after all ops update their pins. This means that the program pictures below, with a 0 constant on the build input pin of a builder, with its build cell being a 1 constant, any ops inputting from the 1 constant will input a `1n`, in the same tick that the constant is replaced by a NIL.

![First tick: 1 constant becomes nil, first 2-wire inputs 1n; Second tick: first wire inputs nil, second wire inputs 1n](docs_imgs/timing.gif?raw=true "First tick: 1 constant becomes nil, first 2-wire inputs 1n; Second tick: first wire inputs nil, second wire inputs 1n")

This timing quirk can be used to propagate signals faster or slower than wires, which is essential as the strict structure of the triangular grid makes precise timing difficult or impossible otherwise. The second exception to simultaneous updating is a necessary consequence of i/os and builders existing, that being handling conflicting operations. If two or more builders build in the same cell on the same turn, or two or more i/os take input or produce output in the same tick, the order in which these conflicting actions occur is left to the interpreter as undefined behavior. However, the following behaviors are implemented in the base interpreter as options:

- Quick: Depends on the order in which ops are updated internally, which is in turn based on the order in which the ops were placed (due to the behavior of JS's `Map`)
- Random: Conflicting operations will be processed in a random order
- Positional: Conflicting operations will be processed based on a pseudorandom priority assigned to each cell (so behavior will always be consistent between runs, but not if the program is moved or rotated)
- Dir. priority: Conflicting build operations will be deconflicted based on which pin of the build cell they are on, while I/O will be ordered roughly from top left to top right

It is recommended that, at minimum, some variation of each of these four be included in any interpreters. Random is important, as it is the only way of generating random numbers (aside from taking them as user input). Positional allows programs to make use of conflicting operations, by either building around the priorities in a pre-chosen spot, or by searching for a place to build a program which has the desired priorities. Dir. priority allows for consistent and easily predictable behavior.

## Halting

Trianguish on its own will never halt, as ops continue to be updated indefinitely. However, the base interpreter comes with an option, "stop on finish", which allows programs to halt, and be treated like those in traditional languages. The mechanism which the base interpreter uses to decide when (if at all) to automatically halt will wait for a turn in which all of the following are true:

1. No ops' IDs or pin states changed
2. No non-exhausted input was taken
3. No output was produced

This is not a perfect heuristic of when a program is "finished" with what it is designed to do, as things like timing loops may continue to run, so programs which intend to halt automatically may require additional logic to disable infinite loops or counters.

## Program storage and byte counts

Trianguish does not lend itself well to being stored in traditional files, given its 2d nature, triangular rather than square grid, base-6-centric numbering, and optional arbitrarily sized integers. The base interpreter comes with a set of functions, `grid_comp`, `grid_dcomp`, `push_param`, and `param`, which respectively: encode the op IDs and positions of a program; decode this representation; encode op IDs, input, and various options into a URL-safe string; and decode this string. As a program is independent of its input and the options it is designed to be run with (which can be thought of as different interpreters), encoding the nonnegative integer returned by `grid_comp` with bijective base 256 is a reasonable, though not necessarily maximally compact, method of storing and counting the bytes of a Trianguish program, and the base interpreter provides functionality (through Ctrl + S, for a file, or Ctrl + Shift + S, for a hexdump of the file in the console) to do this.

## Interpreter controls

The base interpreter works with both keyboard and mouse controls. I find the mouse ones more convenient, but on a laptop with a touchpad the keyboard ones aren't awful.

**Keyboard:**

You can navigate around using WASD or your arrow keys. Use Q/E or dot/comma to rotate between op types and variants, or 2–0 on your keyboard to quickly place certain ops (1 will place a 2-wire, splitter, or s-switch in whatever orientation it thinks will be most useful). You can use Z/X/C or brackets/backslash to change the orientation of ops. To start/stop your program, use R/T/Y/U. You can hold shift with many commands to do a similar or opposite action (with R, it will stop running, but keep any changes made, including by builders).

**Mouse:**

You can left click to change which op is selected for keyboard actions. Left-click-dragging will move the screen, and double left clicking will center the screen around an op. Right click will open an op chooser screen, which also shows certain information about the op, and holding shift while right clicking will allow you to browse the choices to read their descriptions. Right-click-dragging allows you to copy an op from one cell to another. The mouse wheel can be used to rotate ops, or if right click is held down, scroll through the various op types and variants. Clicking the middle mouse button will flip the chirality of the op, and middle-click-dragging will automatically fill in 2-wires, splitters, and s-switches to easily connect things.

**Warning: Changes made when in run mode will not persist after returning to build mode (unless you choose to "stop sim \[and] copy ops")**
