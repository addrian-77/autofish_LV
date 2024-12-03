# Autofish LV - SF
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
- **/setdelay [min] [max]** pentru a seta un delay (in milisecunde) de la intrarea in zona la folosirea comenzii /fish... (if you know you know)
- **/bizdelay [ms]** pentru a seta un delay (in milisecunde) pentru intrarea in magazin

### Instalare

Pentru instalare descarcati si copiati **auto.fish.cs** in folderul **CLEO** (Ignorati fisierele Source code, sunt generate automat)

<details>
<summary>Codul sursa:</summary>

```cpp
{$CLEO}
{$USE CLEO+}
{$INCLUDE SF}
{$USE bitwise}

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

chatmsg "{%x}Auto Fish LV - SF {%x}mod by {%x}iAdriaN {%x}was loaded, {%x}[/setdelay <ms>]{%x}." -1 __WHT __BLU __WHT __BLU __YLW __BLU
chatmsg "{%x}Use {%x}[/autofish] {%x}to toggle the mod {%x}ON {%x}or {%x}OFF." -1 __GRA __YLW __GRA __GRN __GRA __RED             
chatmsg "{%x}Use {%x}[/togdriver] {%x}to toggle auto animations {%x}ON {%x}or {%x}OFF." -1 __GRA __YLW __GRA __GRN __GRA __RED   

// Register the custom command /setdelay
0B34: samp register_client_command "setdelay" to_label @set_delay
0B34: samp register_client_command "bizdelay" to_label @biz_delay

// Register the custom command /autofish
0B34: samp register_client_command "autofish" to_label @auto_fish

// Register the custom command /togdriver
0B34: samp register_client_command "togdriver" to_label @tog_driver

int var_enabled = 1 // Default value, the mod is turned on
int var_storeArea = 0 // Player not in store area
int var_isDriver = 1 // Default value, not a driver
int var_enteredStore = 0
int var_not_used
int var_sphere1
int var_sphere2
int var_sphere3
int var_sphere4
int var_delay = 0 // Default delay set to 0ms
int var_mindelay = 0
int var_maxdelay = 0
int var_bizdelay = 0
int var_aux
float var_posX     
float var_posY     
float var_posZ
longstring var_param  
03BC: var_sphere1 = create_sphere_at 1693.9500 2207.9600 11.0692 radius 2.0
03BC: var_sphere2 = create_sphere_at -1789.8600 1338.6600 7.1797 radius 2.0
03BC: var_sphere3 = create_sphere_at 2637.6101 1129.2800 11.1797 radius 2.0
03BC: var_sphere4 = create_sphere_at -31.0246 -91.3283 1003.5469 radius 2.0        

while true
    wait 0    

    if and
    var_enabled == 1
    var_enteredStore == 0 
    then
        if or
        00ED: actor $PLAYER_ACTOR sphere 0 near_point 1693.9500 2207.9600 radius 0.75 0.75 on_foot
        00ED: actor $PLAYER_ACTOR sphere 0 near_point -1789.8600 1338.6600 radius 0.75 0.75 on_foot
        00ED: actor $PLAYER_ACTOR sphere 0 near_point 2637.6101 1129.2800 radius 0.75 0.75 on_foot
        then
            wait var_bizdelay
            while true
                if or
                00ED: actor $PLAYER_ACTOR sphere 0 near_point 1693.9500 2207.9600 radius 0.75 0.75 on_foot
                00ED: actor $PLAYER_ACTOR sphere 0 near_point -1789.8600 1338.6600 radius 0.75 0.75 on_foot
                00ED: actor $PLAYER_ACTOR sphere 0 near_point 2637.6101 1129.2800 radius 0.75 0.75 on_foot
                then  
                    0B56: set_game_key 15 state 255
                    wait 1
                    0B56: set_game_key 15 state 0
                    wait 100
                    var_enteredStore = 1
                    break
                end
            end
            while var_enteredStore == 1
                wait 0
                if var_enabled == 0
                then
                    break
                end
                if
                00ED: actor $PLAYER_ACTOR sphere 0 near_point -31.0246 -91.3283 radius 1.0 1.0 on_foot
                then
                    0B56: set_game_key 15 state 255
                    wait 1
                    0B56: set_game_key 15 state 0
                    break
                end  
            end
            wait 2000
            0B56: set_game_key 1 state 255
            wait 0
            0B56: set_game_key 1 state 0
        end
    end
    
    if var_enabled == 1
    then
        if Player.Defined($PLAYER_CHAR)
        then
            if not Actor.Driving($PLAYER_ACTOR)
            then
                // Get player position
                00A0: store_actor $PLAYER_ACTOR position_to var_posX var_posY var_posZ
        
                // Check if the player is in the fish zone, after finishing use /fall for safe transport
                // LV
                if and 
                var_posX > 2012.0725 
                var_posX < 2015.5298 
                var_posY > 1511.0432 
                var_posY < 1539.7081 // min x, max x, min y, max y bounradies
                then
                    0209: var_delay = random_int_in_ranges var_mindelay var_maxdelay
                    wait var_delay // Wait based on the delay set
                    say "/fish" // Execute the /fish command
                    var_enteredStore = 0
                    wait 25000 // Wait 25 seconds, it takes 20 seconds to catch a fish
                    if not Actor.Driving($PLAYER_ACTOR)
                    then
                        if var_isDriver == 0 // Check if the player is a driver and needs animations
                        then
                            say "/fall"
                        end
                    end
                    wait 15000 // Wait 15 more seconds in order not to spam the animation, we won't be back this quick anyway
                end
    
                // SF
                if and 
                var_posX > -1237.0 
                var_posX < -1221.0
                var_posY > 424.49 
                var_posY < 481.21 // min x, max x, min y, max y bounradies
                then
                    0209: var_delay = random_int_in_ranges var_mindelay var_maxdelay
                    wait var_delay // Wait based on the delay set
                    say "/fish" // Execute the /fish command
                    var_enteredStore = 0
                    wait 20000 // Wait 15 more seconds in order not to spam the animation, we won't be back this quick anyway
                end
                
                // Check if the player entered the store area, to cancel the animation
                // Only for LV
                if var_isDriver == 0 // Check if the player is a driver or not
                then
                    if and 
                    var_posX > 1679.3840 
                    var_posX < 1716.7549 
                    var_posY > 2184.2734 
                    var_posY < 2210.0962 // min x, max x, min y, max y boundaries
                    then
                        if not Actor.Driving($PLAYER_ACTOR)
                        then
                            if var_storeArea == 0 // Check if the store was visited so that we don't spam the /stop command. Using a wait delay is not viable here
                            then
                                0209: var_delay = random_int_in_ranges var_mindelay var_maxdelay
                                if var_delay == 0 // Use a delay, either the set one or the default 1 second
                                then
                                    wait 1000
                                end
                                say "/stop"
                                
                                var_storeArea = 1 // Mark the store as visited
                            end
                        end
                    end
                
                    // Use the animation again when the player exits the store area
                    if and 
                    var_posX > 1679.3840 
                    var_posX < 1716.7549 
                    var_posY > 2175.3044 
                    var_posY < 2184.2734 // min x, max x, min y, max y boundaries
                    then
                        if not Actor.Driving($PLAYER_ACTOR)
                        then
                            if var_storeArea == 1 // If the store was visited previously, it means that the player just left and needs to use the animation again
                            then
                                0209: var_delay = random_int_in_ranges var_mindelay var_maxdelay
                                if var_delay == 0 // Use a wait delay, either the set one or the default 1 second
                                then
                                    wait 1000
                                else
                                    wait var_delay
                                end
                                say "/fall"            
                                var_storeArea = 0 // Mark the store as not visited 
                                var_enteredStore = 0
                            end
                        end
                    end  
                end
                 
            end
        end
    end
end

:set_delay
if 0B35: samp var_param = get_last_command_params
then
    var_delay = 0
    if 0AD4: var_not_used = scan_string var_param format "%d %d" var_mindelay var_maxdelay // Scan for a single integer after "/setdelay"
    then
        if var_mindelay > var_maxdelay
        then
            var_aux = var_mindelay
            var_mindelay = var_maxdelay
            var_maxdelay = var_aux
        end 
        chatmsg "{%x}<{%x}!!!{%x}> {%x}Min-max delay set to: {%x}%d {%x}- {%x}%d {%x}(miliseconds)" -1 __RED __WHT __RED __WHT __YLW var_mindelay __WHT __YLW var_maxdelay __WHT // Show confirmation message
    else
        chatmsg "{%x}<{%x}!!!{%x}> {%x}Error: invalid values" -1 __RED __WHT __RED __WHT // Show error if no number is provided
    end
else
    chatmsg "{%x}<{%x}!!!{%x}> {%x}Syntax: {%x}[/setdelay <min> <max>]" -1 __RED __WHT __RED __WHT __YLW
end
SAMP.CmdRet()

:biz_delay
if 0B35: samp var_param = get_last_command_params
then
    var_delay = 0
    if 0AD4: var_not_used = scan_string var_param format "%d" var_bizdelay // Scan for a single integer after "/setdelay"
    then 
        chatmsg "{%x}<{%x}!!!{%x}> {%x}Business enter delay set to: {%x}%d {%x}(miliseconds)" -1 __RED __WHT __RED __WHT __YLW var_bizdelay __WHT // Show confirmation message
    else
        chatmsg "{%x}<{%x}!!!{%x}> {%x}Error: invalid value" -1 __RED __WHT __RED __WHT // Show error if no number is provided
    end
else
    chatmsg "{%x}<{%x}!!!{%x}> {%x}Syntax: {%x}[/bizdelay <ms>]" -1 __RED __WHT __RED __WHT __YLW
end
SAMP.CmdRet()

:auto_fish
if var_enabled == 1
then
    var_enabled = 0
    chatmsg "{%x}Auto Fish {%x}disabled" -1 __WHT __RED
    03BD: destroy_sphere var_sphere1
    03BD: destroy_sphere var_sphere2
    03BD: destroy_sphere var_sphere3
    03BD: destroy_sphere var_sphere4
else
    var_enabled = 1
    var_enteredStore = 0
    chatmsg "{%x}Auto Fish {%x}enabled" -1 __WHT __GRN
    03BC: var_sphere1 = create_sphere_at 1693.9500 2207.9600 11.0692 radius 2.0
    03BC: var_sphere2 = create_sphere_at -1789.8600 1338.6600 7.1797 radius 2.0
    03BC: var_sphere3 = create_sphere_at 2637.6101 1129.2800 11.1797 radius 2.0
    03BC: var_sphere4 = create_sphere_at -31.0246 -91.3283 1003.5469 radius 2.0       
end
SAMP.CmdRet()

:tog_driver
if var_isDriver == 0
then
    var_isDriver = 1
    chatmsg "{%x}<{%x}!!!{%x}> {%x}You are now a {%x}driver{%x}, animations won't be applied" -1 __WHT __RED __WHT __GRA __RED __GRA
else
    var_isDriver = 0
    chatmsg "{%x}<{%x}!!!{%x}> {%x}You are not a {%x}driver{%x} anymore, animations will be applied" -1 __WHT __RED __WHT __GRA __RED __GRA
end
SAMP.CmdRet()
```
</details>
