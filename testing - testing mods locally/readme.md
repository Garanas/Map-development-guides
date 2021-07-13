### Introduction
A recent problem has risen in which mods are repeatedly uploaded to the vault for testing purposes. In turn, the vault is repeatedly updated by authors that make only minor changes. Generally the testing happens without any additional players - therefore the testing could also be performed locally.

This article attempts to deal with this issue as it is not required for the majority of mods to be uploaded for testing purposes. In turn, this will prevent:
 - Versions of your mod taking up space on the server, while never being downloaded / used.
 - And as a bonus: feeding of replays of testing sessions to the replay server.

### Requisites
I assume that you are familiar with making mods. This article does not explain to you how to make or edit mods.

### Steps to testing mods locally

#### Where are the mods stored on your hard drive
All mods that you use are stored locally on your hard drive. They can generally be found at the following location:

_C:/Users/%USERPROFILE%/Documents/My Games/Gas Powered Games/Supreme Commander Forged Alliance/mods_

Replace '%USERPROFILE% with your user profile. If the 'mods' folder does not exist then you can create one. Store your mod, as any other mod, in this folder. 

#### Starting the game without any client
The FAF client stores a lot of data on your hard drive. This includes the latest patches to the game, various caches and featured mods such as the Nomads mod. It can generally be found at the following location:

_C:/ProgramData/FAForever_

What is especially interesting is the 'bin' folder that contains the executable 'ForgedAlliance.exe'. Double click this executable to launch the game with the patches provided by the FAF community. Any session you run in this instance will not influence any of the online services that FAF provides, such as the replay vault.

You can start a skirmish session and select the mod(s) you wish to test.

#### Small introduction on the development cycle

A few tips with regard to the development cycle:
 - When you change the _%MOD%\_info.lua_ file you need to restart the game.
 - When you restart a map the mod is reloaded and all changes are applied.

You can open up the moho log as usual with 'F9' or by searching for 'toggles the debugging window' in the Key Bindings menu (F1).

You can let the game re-load any disk changes that you make in the mounted (accessible) folders of the game. You can do so by adding /EnableDiskWatch to the list of arguments when launching the game. This can be achieved via creating a shortcut to the executable and then adding /EnableDiskWatch to its target at the end. As an example:

_C:/ProgramData/FAForever/bin/ForgedAlliance.exe /EnableDiskWatch_

#### Where to start modding
There are numerous sources out there to help you get going. As an example:
 - https://supcom.fandom.com/wiki/Beginners_Tutorial._SC/FA_Creating_an_Upgradeable_Structure.
 - https://supcom.fandom.com/wiki/Category:Mod_creation

And a visual guide:
 - https://www.youtube.com/watch?v=SYfb_XhH25s

Tutorials with regard to mods are rather scattered at the moment. Feel free to contribute and assist in cleaning it all up. This article is by no means an introduction to making mods.

### FAQ

#### I have the vault fallback location enabled - does that influence anything?
No - both locations (the original one mentioned in this article) and the fallback location is mounted (accessible) for the game, allowing you to use either location.