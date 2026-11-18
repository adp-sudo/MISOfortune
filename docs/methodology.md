# Methodology

All four of us worked in the same rhythm, even though the projects look different on the surface. This page describes that rhythm and then shows how it played out in each module.

## The shared workflow

**1. Decide what the master needs to know.** Before any Verilog, we wrote down what each slave has to report and what the master is allowed to command. This decides what travels on MOSI and what comes back on MISO, and it keeps the protocol small.

**2. Draw the state machines.** Slaves are finite state machines, and so is the master wherever it makes decisions. Drawing them first made the Verilog almost mechanical to write.

**3. Build bottom up.** A shift-register level SPI core first, then slave behaviour on top, then the master that coordinates everything. Each layer could be tested before the next one existed.

**4. Test each piece alone, then together.** Module testbenches come first. After that comes an integration testbench where the real master talks to all the slaves at once. Most of our bugs showed up only at this stage, which is why it is worth doing.

**5. Make the testbench check itself.** Wherever we could, the testbench compares what it received against what it expected and reports pass or fail. Reading waveforms by eye is fine for debugging but not for confidence.

**6. Wrap it in a scenario.** The mine rescue, the factory and the drone are there because a story makes the behaviour easy to explain, and because it gave each of us concrete rules to verify.

## How it played out in each module

### Time Loop

The design starts at the bottom: the PHY, which only knows how to clock bits in and out under a given mode and timing. The scheduler sits above it and only knows how to start one transfer per device and wait for it to finish. The top level adds the three simulated devices and the alert rules. Testbenches exist for each layer. The PHY testbench checks the same MOSI and MISO bytes in all four modes, and the integration testbench runs a safe scenario plus three alert scenarios by creating parameterised copies of the design.

### Cotton Factory

The key design decision was to put the rule in the master. Each slave is a simple batch counter with a request to rest. The master polls all three, and the invariant that at least one machine is always active is enforced in one place, at the moment rest is granted. Verification is simulation based, using a testbench that runs the three machines through repeated cut and rest cycles.

### Drone Strike

Here the rule enforcement lives in the slaves. The target slave only accepts a hit when it is in the locked state, and it only starts locking if the aim is within one cell of the real target. The master and the HUD stay simple. The Verilog testbench prints each state next to the expected one and writes a trace file, and the same trace can be replayed in the Python HUD, so the simulation and the visual demo are guaranteed to agree.

### Nivetha's module

To be added.

## Tools

Vivado behavioural simulation for all Verilog, Python with pygame for the drone HUD, and Git and GitHub for sharing the work.
