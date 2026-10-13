# Task 2: Connecting the counter to Vbuddy

## What I implemented
I extended the counter testbench to communicate with Vbuddy by including `vbuddy.cpp`.
The counter is still simulated on my computer using Verilator. Vbuddy provides a physical display and a push-button input; the counter circuit is not uploaded to the board.

The testbench opens the connection using `vbdOpen()` and sets the screen header with `vbdHeader("Lab 1: Counter")`. The serial-device path is specified in `vbuddy.cfg`.

## How the display works
The testbench reads `top->count` and sends its value to Vbuddy using `vbdHex()`.
Each call displays one hexadecimal digit, with position 1 being the rightmost digit.

```cpp
vbdHex(2, (int(top->count) >> 4) & 0xF);
vbdHex(1, int(top->count) & 0xF);
```

An 8-bit count is made up of two groups of four bits. `& 0xF` selects the lowest four bits, while `>> 4` shifts the upper four bits into that position. For example, a count of 58 in decimal is `3A` in hexadecimal, so the display shows `3A`.

My testbench also writes positions 3 and 4. These display zero because the count is only 8 bits wide. `vbdCycle(i+1)` displays the simulation cycle number separately from the counter value.

## How the button controls the counter

```cpp
top->en = vbdFlag();
```

In toggle mode, pressing the rotary encoder button changes the stored flag between 0 and 1. `vbdFlag()` reads that flag and assigns it to the counter's enable input.

- When reset is low and the flag is 1, the counter increments on each rising clock edge.
- When reset is low and the flag is 0, the counter holds its value.
- Reset takes priority over enable and clears the count on the next rising edge.

The enable assignment comes after the clock-toggle loop, so it controls the next cycle. The calls to `top->eval()` in that next cycle evaluate the circuit using the new input.

## Testing and demonstration
The testbench runs for 300 clock cycles and records the signals in `counter.vcd`. It sends the count to Vbuddy after each complete clock cycle.

[Counter demonstration video](video/counter.mp4)

The current testbench asserts reset initially and again after `i == 15`, so that later reset should be considered when interpreting the results.

## CHALLENGE YOURSELF
### Plotting the count
The next exercise is to replace the hexadecimal-display calls with:

```cpp
vbdPlot(int(top->count), 0, 255);
```

This plots the count on the TFT screen. Counting should produce an increasing plot, while disabling the counter should produce a flat section. This change is not yet present in my current testbench.

### Up/down counter
For the challenge, `en` changes meaning: 1 selects counting up and 0 selects counting down, instead of holding. The button can still control this input through `vbdFlag()`.

My current counter still implements enable/hold behavior. After changing it, I should check both directions and wrapping from 255 to 0 and from 0 to 255.

## Final cleanup
The current testbench still needs `vbdClose();` after the outer loop to close the board connection normally before finishing.

## Commands used
Run these commands from the `task2` folder with Vbuddy connected and its USB device attached to Ubuntu:

```bash
verilator -Wall --cc --trace counter.sv --exe counter_tb.cpp
make -j -C obj_dir/ -f Vcounter.mk Vcounter
obj_dir/Vcounter
gtkwave counter.vcd
```
