# Ping-Pong-FPGA
![Screenshot 2024-03-04 204944](https://github.com/MichalisTsak/Ping-Pong-FPGA/assets/104669614/2d08bf5d-053a-41f7-a166-eb663a6301d9)


This project implements a simple Ping Pong game using SystemVerilog, targeting an  Nexys A7-100T Artix 7 FPGA. The game features two player-controlled paddles and a bouncing ball, 
with the objective of keeping the ball within play to score points. The display, paddle movement, and scoring logic are all implemented in SystemVerilog.
So by connecting the FPGA with a VGA to a computer screen two players could play the game. 

**Note**: The resolution is 800x600 at 72Hz refresh rate. So for each line we spend 800+240=1040 cycles of the pixel clock and the total number of lines to count is 600+37+6+23=666 lines.
So basically the total number of pixel clock periods needed for a frame is 1040 pixel clocks/line x 666 line/frame = 692640 pixel clocks. 
Since we want to display 72 frames per second (refresh rate = 72Hz), then 692640 pixel clocks/frame x 72 frames/sec = 49870080 pixel clocks/sec i.e. 1 pixel every 20.05nsec. 
In order for our system to support sending this amount of data, it must be running on a clock with a period of 20.05nsec i.e. 50MHz, as shown in the seventh row of the second column of the table.
But our FPGA has a clock of 100MHz. Therefore, a divider of the clock frequency from 100MHz to 50MHz was created.


![Screenshot 2024-03-04 213214](https://github.com/MichalisTsak/Ping-Pong-FPGA/assets/104669614/f52641e3-706d-41e5-917f-11480a165dcc)

# Modules:

## PanelDisplay:

Manages the overall game display, including ball movement, player paddles, scoring, and synchronization signals.
Utilizes clock generation, edge detection, and continuous updates for game logic.
Outputs RGB color signals, synchronization signals, and a 12-bit score.

## ArcadeGameMovement:

Handles the movement of player paddles based on user input (button presses).
Uses a two-flop synchronizer (two_flop_sync) to debounce and synchronize button inputs.
Outputs signals (move_up, move_down) indicating the desired direction of paddle movement.

## edge_detector:

the update of the limits must occur once per frame, i.e. it must it must be checked that within the time span, where vsync is 0, the boundary update will occur once
once. That is, it is not enough to only check if vsync is 0, as vsync is at a reasonable zero  for more than 1 cycle, and this will result in multiple changes to your limits.

![Screenshot 2024-03-04 212544](https://github.com/MichalisTsak/Ping-Pong-FPGA/assets/104669614/5a822733-130a-4f49-8282-3452ae93c463)

In order to achieve this you will need to use a negative edge detector which will ensure that the boundary refresh only happens once per frame when vsync is zero. 
The purpose of move_enable is to allow changing the boundaries of the center square
## two_flop_sync:

Implements a two-flop synchronizer to debounce and synchronize input signals.

**Debouncing**: The two-flop structure helps eliminate noise and bouncing effects in the input signal. Rapid changes in the input signal are filtered out, providing a stable and synchronized output.
Use in ArcadeGameMovement:

two_flop_sync is employed in ArcadeGameMovement to process button inputs (P_Up and P_Dn). By synchronizing these signals, the module ensures that button presses and releases are reliably detected, reducing the likelihood of false triggers due to noise.

## Reference picture for pixels 
![Screenshot 2024-03-04 211718](https://github.com/MichalisTsak/Ping-Pong-FPGA/assets/104669614/875eff90-c1fc-41ce-a337-28a9eee86fc4)


