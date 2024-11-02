# Autofish LV
**Pentru a descarca modul, apasati pe butonul <>Code, iar apoi pe Download ZIP**
#

> ### !DISCLAIMER!
> Modul a fost conceput pentru a face fish cu mai multe conturi in acelasi timp. Este o automatizare partiala, deoarece tot trebuie sa vinzi pestele manual de pe toate conturile.

Acesta este un mod **CLEO** open-source care automatizeaza procesul de fish. Modul da **/fish** automat cand intrati in zona din afara barcii.

### **Lista cu comenzi:**

- **/autofish** pentru a activa sau dezactiva modul
- **/togdriver** pentru a aplica animatii
  <details>
  <summary>(click to expand)</summary>
  Modul va folosi automat animatia /fall pentru conturile care stau pe capota, va opri animatia cand ajungeti la 24/7 de langa PNS si o va porni iar la plecare. Daca sunteti sofer nu aveti nevoie de animatii, deci trebuie sa dezactivati functia folosind comanda.
  </details>
- **/setdelay [ms]** pentru a seta un delay (in milisecunde) de la intrarea in zona la folosirea comenzii /fish... (if you know you know)

### Instalare

Pentru instalare descarcati si copiati **auto.fish.cs** in folderul **CLEO** (Ignorati fisierele Source code, sunt generate automat)

<details>
<summary>Codul sursa:</summary>

```cpp
{$CLEO}
{$INCLUDE SF}

0000:

/*
    Auto Fish LV mod with custom delay
    Credits to Krystian and ChatGPT
*/

repeat
wait 0
until SAMP.Available()

const
    __BLU = 0x268efc
    __RED = 0xff2b2b
    __YLW = 0xffeb0f
    __WHT = 0xffffff
    __GRN = 0x4ae072
    __GRA = 0xcfcfcf
end

chatmsg "{%x}Auto Fish LV {%x}mod by {%x}iAdriaN {%x}was loaded, {%x}[/setdelay <ms>]{%x}." -1 __WHT __BLU __WHT __BLU __YLW __BLU
chatmsg "{%x}Use {%x}[/autofish] {%x}to toggle the mod {%x}ON {%x}or {%x}OFF." -1 __GRA __YLW __GRA __GRN __GRA __RED             
chatmsg "{%x}Use {%x}[/togdriver] {%x}to toggle auto animations {%x}ON {%x}or {%x}OFF." -1 __GRA __YLW __GRA __GRN __GRA __RED

// Register the custom command /setdelay
0B34: samp register_client_command "setdelay" to_label @set_delay
3@ = 0 // Default delay set to 0ms
5@ = 0

// Register the custom command /autofish
0B34: samp register_client_command "autofish" to_label @auto_fish
6@ = 1 // Default value, the mod is turned on
7@ = 0

10@ = 0 // Player not in store area

// Register the custom command /togdriver
0B34: samp register_client_command "togdriver" to_label @tog_driver
11@ = 0 // Default value, not a driver
8@ = 0 // Command not initialized
:set_delay
if SAMP.IsCommandTyped(0@)
then
    if 0AD4: $NOT_CMD = scan_string 0@ format "%d" 3@ // Scan for a single integer after "/setdelay"
    then
        chatmsg "{%x}<{%x}!!!{%x}> {%x}Delay set to: %d ms" -1 __RED __WHT __RED __WHT 3@ // Show confirmation message
    else
        chatmsg "{%x}<{%x}!!!{%x}> {%x}Error: invalid value" -1 __RED __WHT __RED __WHT // Show error if no number is provided
    end
else
    if 5@ == 1
    then
        chatmsg "{%x}<{%x}!!!{%x}> {%x}Syntax: {%x}[/setdelay <ms>]" -1 __RED __WHT __RED __WHT __YLW
    else
        5@ = 1
    end
end
SAMP.CmdRet()

:auto_fish
if 7@ == 1
then
    if 6@ == 1
    then
        6@ = 0
        chatmsg "{%x}Auto Fish {%x}disabled" -1 __WHT __RED
    else
        6@ = 1
        chatmsg "{%x}Auto Fish {%x}enabled" -1 __WHT __GRN
    end
else
    7@ = 1
end
SAMP.CmdRet()

:tog_driver
if 8@ == 1
then
    if 11@ == 0
    then
        11@ = 1
        chatmsg "{%x}<{%x}!!!{%x}> {%x}You are now a {%x}driver{%x}, animations won't be applied" -1 __WHT __RED __WHT __GRA __RED __GRA
    else
        11@ = 0
        chatmsg "{%x}<{%x}!!!{%x}> {%x}You are not a {%x}driver{%x} anymore, animations will be applied" -1 __WHT __RED __WHT __GRA __RED __GRA
    end
else
    8@ = 1
end
SAMP.CmdRet()

:main_loop
wait 0

if 6@ == 1
then
    if Player.Defined($PLAYER_CHAR)
    then
        if not Actor.Driving($PLAYER_ACTOR)
        then
            // Get player position
            00A0: store_actor $PLAYER_ACTOR position_to 0@ 1@ 2@
    
            // Check if the player is in the fish zone, after finishing use /fall for safe transport
            if and 
            0@ > 2012.0725 
            0@ < 2015.5298 
            1@ > 1511.0432 
            1@ < 1539.7081 // min x, max x, min y, max y bounradies
            then
                wait 3@ // Wait based on the delay set
                say "/fish" // Execute the /fish command
                wait 25000 // Wait 25 seconds, it takes 20 seconds to catch a fish
                if not Actor.Driving($PLAYER_ACTOR)
                then
                    if 11@ == 0 // Check if the player is a driver and needs animations
                    then
                        say "/fall"
                    end
                end
                wait 15000 // Wait 15 more seconds in order not to spam the animation, we won't be back this quick anyway
            end

            
            // Check if the player entered the store area, to cancel the animation
            if 11@ == 0 // Check if the player is a driver or not
            then
                if and 
                0@ > 1679.3840 
                0@ < 1716.7549 
                1@ > 2184.2734 
                1@ < 2210.0962 // min x, max x, min y, max y boundaries
                then
                    if not Actor.Driving($PLAYER_ACTOR)
                    then
                        if 10@ == 0 // Check if the store was visited so that we don't spam the /stop command. Using a wait delay is not viable here
                        then
                            if 3@ == 0 // Use a delay, either the set one or the default 1 second
                            then
                                wait 1000
                            end
                            say "/stop"
                            10@ = 1 // Mark the store as visited
                        end
                    end
                end
            
                // Use the animation again when the player exits the store area
                if and 
                0@ > 1679.3840 
                0@ < 1716.7549 
                1@ > 2175.3044 
                1@ < 2184.2734 // min x, max x, min y, max y boundaries
                then
                    if not Actor.Driving($PLAYER_ACTOR)
                    then
                        if 10@ == 1 // If the store was visited previously, it means that the player just left and needs to use the animation again
                        then
                            if 3@ == 0 // Use a wait delay, either the set one or the default 1 second
                            then
                                wait 1000
                            else
                                wait 3@
                            end
                            say "/fall"            
                            10@ = 0 // Mark the store as not visited 
                        end
                    end
                end
            end
            
        end
    end
end

jump @main_loop
```
</details>
