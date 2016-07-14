# Incisive Verilog options for modules with same names

The design hierarchy is as follows:

```
a
|- b
|  |- c
|     |- d
|- c
   |- d
```

Say `b` is the top-level design under test and `a` and is the top-level testbench. `b, c, d` are synthesized to `*.syn.v`.
We want to simulate with RTL versions of `a, c, d` in the testbench portion of the hierarchy and synthesized versions of `b, c, d` in the design portion.

In the `lib.map` script, the `library` line specifies that the synthesized verilog files should compile to a library called `synlib` (the rest go to the default `worklib`). The config block controls the binding of modules in libraries to specific instances in the design. By default, all instances are taken from `worklib`. However, the instance of `b` and all its descendants should be taken from `synlib`.

Run irun as:

```
irun -libverbose *.v -libmap lib.map -top cfg
```

and examine the compiler messages. In particular, at the beginning there should be lines like:

```
file: b.syn.v
        module synlib.b:v
```

that say which library each file goes into (library mapping during compile). After that, there should be messages like:

```
Resolved design unit 'b' at 'a.u_b' to 'synlib.b:v' (using Verilog configuration).
```

that say which library each instance is taken from (binding during elaboration).

## Extra

In cdnshelp, search for `-libmap` to see what else can go in the file. For example, you can supply a list of libraries after `liblist` instead of just one as shown in this example. During binding, the first library in the list gets priority over the next and so on.

Also look at the options `-filemap/-endfilemap` and `-makelib/-endlib`. `makelib` can be used in place of the `library` line in lib.map:

```
irun -libverbose a.v c.v d.v -makelib synlib *.syn.v -endlib -libmap lib.map -top cfg
```

The syntax of lib.map is specified in Verilog 2001 standard (IEEE Standard 1364-2001 Version C, Chapter 13 "Configuring the contents of a design"). This feature should be supported by other simulators as well.

However, this feature cannot be used with the older `-v, -y` options of specifying source files and folders as libraries.
