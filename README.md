# run python component by rust host

first step：pip install componentize-py
```
package example:component;
world example {
    export add: func(x: s32, y: s32) -> s32;
}
```


secend step：create a add.wit and execute componentize-py --wit-path add.wit --world example bindings .

the third：use app.py to implement the add.wit
import example
```
class Example(example.Example):
    def add(self, x: int, y: int) -> int:
        return x + y
```     
 fourth：generate component
 componentize-py --wit-path add.wit --world example componentize app -o add.wasm
 
 fifth：use rust host to load and execute component https://github.com/bytecodealliance/component-docs/blob/main/component-model/examples/example-host/src/main.rs

 git clone https://github.com/bytecodealliance/component-docs.git && cd component-docs/component-model/examples/example-host
 rm -rf Cargo.lock && rm -rf target
 cargo run --release -- 1 2 ../../../../add.wasm
 
 
 
 print 1 + 2 = 3
 
 

# Building a Calculator of Wasm Components

This tutorial walks through how to compose a component to build a Wasm calculator.
The WIT package for the calculator consists of a world for each mathematical operator
add an `op` enum that delineates each operator. The following example interface only
has an `add` operation:

```wit adder
package docs:adder@0.1.0;


interface add {
    add: func(a: u32, b: u32) -> u32;
}

world adder {
    export add;
}
```


```wit calculator
package docs:calculator@0.1.0;

interface calculate {
    enum op {
        add,
    }
    eval-expression: func(op: op, x: u32, y: u32) -> u32;
}

world calculator {
    export calculate;
    import docs:adder/add;
}

world app {
    import calculate;
}
```


To expand the exercise to add more components, add another operator world, expand the enum, and modify the `command` component to call it.

## Building and running the example

To compose a calculator component with an add operator, run the following:

```sh
(cd calculator && cargo component build --release)
(cd adder && cargo component build --release)
(cd command && cargo component build --release)
cd ..
wasm-tools compose calculator/target/wasm32-wasip1/release/calculator.wasm -d adder/target/wasm32-wasip1/release/adder.wasm -o composed.wasm
wasm-tools compose command/target/wasm32-wasip1/release/command.wasm -d composed.wasm -o final.wasm
```

Now, run the component with wasmtime:

```sh
wasmtime run final.wasm 1 2 add
1 + 2 = 3
```
