

### Parts

### Visual Studio Code

### Original scripts

### FAF scripts

### Concept: hooking into scripts

### Mod scripts

### Concept: non-destructive hooking

### Map scripts

There is a guide on writing mission scripts on the Wikipedia:
 - https://wiki.faforever.com/index.php?title=Mission_Scripting

Luckily for us, all the LUA code is visible in plaintext. A nice consequence of this is that we can open up any of the campaign maps and look at their scripts. These can be found at:
_%to-steam%\steam\steamapps\common\supreme commander\maps_
_%to-steam%\steam\steamapps\common\supreme commander forged alliance\maps_

The campaign maps in question are _X1CA_00X_ for Forged Alliance and _SCCA_A0X_, _SCCA_E0X_ and _SCCA_R0X_ for the original game. 

These maps, in combination with the source files, provide a great place to start scripting. As an example, say a mission script imports the following file:

``` lua
local ScenarioFramework = import('/lua/scenarioframework.lua')
local ScenarioUtils = import('/lua/sim/ScenarioUtilities.lua')
```

By viewing their import path you can see where they are stored. In turn, you can find them in the script files and open them up. This will help with your understand as to what is possible and how you can achieve this.
