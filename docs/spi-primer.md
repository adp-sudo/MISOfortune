# SPI primer

This is a short refresher for anyone reading the repository without a protocol background.

## The four signals

SCLK is the clock and always comes from the master. MOSI (Master Out, Slave In) carries data from the master to the slave. MISO (Master In, Slave Out) carries data from the slave back to the master. CS, also written SS, is the chip select. It is active low, so a slave listens only while its CS line is pulled down.

With more than one slave, SCLK, MOSI and MISO are shared by everyone and every slave gets its own CS line. The cotton factory uses CS1, CS2 and CS3, and the time loop and drone projects follow the same idea.

## A transfer is a swap

The master and the selected slave each hold a shift register. On every clock pulse one bit leaves the master on MOSI while one bit leaves the slave on MISO. After eight clocks (or sixteen, in the drone project's frames) the two registers have traded contents. This is why a slave in our projects can send back a status byte during the very same transaction in which it receives a command.

Because the slave can only speak while the master is clocking it, the master always has to ask first. That is the root of the MISOfortune idea: any bad news from a slave has to wait for the master to poll it, and then it comes back on MISO.

## The four modes

Two settings decide the timing. CPOL is the idle level of SCLK, and CPHA says whether data is sampled on the first or the second clock edge.

| Mode | CPOL | CPHA | Idle clock | Sampled on        |
| ---- | ---- | ---- | ---------- | ----------------- |
| 0    | 0    | 0    | low        | rising edge       |
| 1    | 0    | 1    | low        | falling edge      |
| 2    | 1    | 0    | high       | falling edge      |
| 3    | 1    | 1    | high       | rising edge       |

Master and slave must agree on the mode or the bits will be sampled at the wrong moment. The cotton factory and the drone project use mode 0. Madhuri's PHY supports all four and her testbench checks each one.

## Where each project fits

| Project        | Slaves | Frame           | Mode        |
| -------------- | ------ | --------------- | ----------- |
| Time Loop      | 3      | 8 bit           | 0 to 3      |
| Cotton Factory | 3      | 8 bit           | 0           |
| Drone Strike   | 4      | 16 bit, MSB first | 0         |
