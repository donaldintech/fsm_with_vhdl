# Implementing a FSM using VHDL

**Finite State Machines(FSMs)**, from a few of my previous tutorials, are models that can be used 
to represent various processes-where there only can exist one state at a time. 

and in this article, we'll explore how we can model a washing machine -but this time,
using VHDL.

# Tools used
For this article, we'll make good use of simulation using Digital, where we can write all our code as well as view our 
process in action.

Another cool resource used is **Brock J. LaMeres' book-Introduction to Logic Circuits & Logic Design with VHDL**

that said let's go!!

# Step 1 : Word Description
This step plays a huge role in dictating what the task is all about, and it being clear and precise makes it easier 
for the implementation of subsequent steps.

```
The design of the washing machine begins in an idle state, waiting for the presence of clothes(C), detergent(D), and water(W). Once
 all are available, it transitions to fill_water state to prepare for washing. The machine then moves through wash, drain,
 rinse, and spin states sequentially, simulating each stage of the washing process. In each state, the appropriate outputs
(In this case L for Clothes load, and U for used water), are set based on the current state. The cycle completes in
 the done state, where the machine returns to idle, ready for the next load.
```
# Step 2: Drawing a State Diagram
Basically a visual representation of the whole machine process, it also plays a role in making inputs and outputs visibly 
placed.

![image](https://github.com/user-attachments/assets/e3ec3390-d2ed-4126-9bab-1d2f1141a129)

Note: the ones in Red are outputs, while Black are inputs

# Step 3: Synthesis of the FSM

There are three main components of a state machine: the **state memory**, the **next state logic**, and the **output
logic**.

At this point then we introduce our VHDL to do our synthesis.

**a.state memory logic**

The state memory holds the current state of the system. The current
state is updated with next state on every rising edge of the clock, which is indicated with the “>” symbol
within the block.

```
 State_Memory : process (Reset, Clk) is
  begin
    if (Reset = '0') then
      current_state <= idle;
    elsif rising_edge(Clk) then
      current_state <= next_state;
    end if;
  end process;
```
**b.next state logic**

The next state logic block is a group of combinational logic that produces the next state signals based on the
current state and any system inputs.

```
Next_State_Logic : process (current_state, C, D, W) is
  begin
    case current_state is
      when idle =>
        if (C = '1' and D = '1' and W = '1') then
          next_state <= fill_water;
        else
          next_state <= idle;
        end if;
        
      when fill_water =>
        next_state <= wash;
        
      when wash =>
        next_state <= drain;

      when drain =>
        next_state <= rinse;

      when rinse =>
        next_state <= spin;

      when spin =>
        next_state <= done;
        
      when done =>
        next_state <= idle;
        
      when others =>
        next_state <= idle;
    end case;
  end process;
```
**c.output logic**

The output logic block is a group of combinational logicthat creates the outputs of the system. 
This block always uses the current state as an input and, depending on the type of machine (Mealy or Moore), uses the 
system inputs.

For our case, the FSM is a **Mealy machine**; hence both current state and system inputs determine the output.

```
Output_Logic : process (current_state) is
  begin
    -- Default Outputs
    L <= '0';
    U <= '0';

    case current_state is
      when wash =>
        L <= '0';
        U <= '0';

      when drain =>
        L <= '1';
        U <= '1';

      when rinse =>
        L <= '1';
        U <= '1';

      when spin =>
        L <= '1';
        U <= '0';

      when done =>
        L <= '1';
        U <= '0';

      when others =>
        L <= '0';
        U <= '0';
    end case;
  end process;
```
# Step 4: Simulation

Merging with our full code, we can now open our Digital Software tool.

![image](https://github.com/user-attachments/assets/a4148cf2-16f1-4f2c-a676-ad6e73693ef3)

You should then be able to cast the whole code in the field given.

After checking to make sure no errors ensue, add outputs in the form of LEDs to represent outputs,
as well as a clock, Reset and the three inputs appropriately.

![image](https://github.com/user-attachments/assets/8066a291-d0bd-4f8d-ae48-9422c4c0fb35)

You can then proceed to simulate and view the magic! That's all.
You can find the full code below:

```
library IEEE;
use IEEE.std_logic_1164.all;

entity washing_FSM is
  port (
    Clk   : in std_logic;
    Reset : in std_logic;
    C, D, W : in std_logic;
    L, U   : out std_logic
  );
end entity;

architecture improved_FSM_arch of washing_FSM is
  type wash_typ is (idle, fill_water, wash, drain, rinse, spin, done);
  signal current_state, next_state : wash_typ;
begin

  -- State Memory Process
  State_Memory : process (Reset, Clk) is
  begin
    if (Reset = '0') then
      current_state <= idle;
    elsif rising_edge(Clk) then
      current_state <= next_state;
    end if;
  end process;

  -- Next State Logic
  Next_State_Logic : process (current_state, C, D, W) is
  begin
    case current_state is
      when idle =>
        if (C = '1' and D = '1' and W = '1') then
          next_state <= fill_water;
        else
          next_state <= idle;
        end if;
        
      when fill_water =>
        next_state <= wash;
        
      when wash =>
        next_state <= drain;

      when drain =>
        next_state <= rinse;

      when rinse =>
        next_state <= spin;

      when spin =>
        next_state <= done;
        
      when done =>
        next_state <= idle;
        
      when others =>
        next_state <= idle;
    end case;
  end process;

  -- Output Logic
  Output_Logic : process (current_state) is
  begin
    -- Default Outputs
    L <= '0';
    U <= '0';

    case current_state is
      when wash =>
        L <= '0';
        U <= '0';

      when drain =>
        L <= '1';
        U <= '1';

      when rinse =>
        L <= '1';
        U <= '1';

      when spin =>
        L <= '1';
        U <= '0';

      when done =>
        L <= '1';
        U <= '0';

      when others =>
        L <= '0';
        U <= '0';
    end case;
  end process;

end architecture;
```

