
### Parts

### Requisites
Basic understanding of a development environment and / or coding.

### Introduction
This article is in its core a re-write of the readme file of the FAF git repository. See also:
 - https://github.com/FAForever/fa

It is part of this series on request to ensure it is complete.

### Cloning or downloading the repository
You can clone or download the FAF repository from the github repository:
 - https://github.com/FAForever/fa

Store the repository in your workspace. If you downloaded the repository then make sure to unpack it before you continue. If everything is alright then in your workspace you should see a folder structure that is roughly equivalent to:

```
├───effects
├───engine
├───env
├───etc
├───loc
├───lua
├───meshes
├───projectiles
├───props
├───schook
├───testmaps
├───textures
└───units
```

### Development initialisation file
Navigate to the FAF program data folder. It can generally be found at the following location:
`C:/ProgramData/FAForever`

Then navigate into the bin folder. You'll find various files in here. I've marked the relevant files:
```
BsSndRpt.exe
BugSplat.dll
BugSplatRc.dll
DbgHelp.dll
ForgedAlliance.exe              -- !
GDFBinary.dll
init-dev.lua                    -- !
init.lua                        -- !
init_claustrophobia.lua         -- !
init_coop.lua                   -- !
init_equilibrium.lua            -- !
init_faf.lua                    -- !
init_fafbeta.lua                -- !
init_fafdevelop.lua             -- !
init_koth.lua                   -- !
init_labwars.lua                -- !
init_ladder1v1.lua              -- !
init_murderparty.lua            -- !  
init_nomads.lua                 -- !
init_nonxt.lua                  -- !  
init_phantomx.lua               -- !
init_xtremewars.lua             -- !
msvcm80.dll
msvcp80.dll
msvcr80.dll
SHSMP.DLL
start-dev-lobby.bat
start-dev-map.bat
start-dev-rainmakers.bat
SupComDataPath.lua
SupComDataPathFAF.lua
sx32w.dll
tree.txt
wxmsw24u-vs80.dll
zlibwapi.dll
```

The executable is the C++ part of the game that we cannot edit in practice. The initialisation files tell the game where to look for various files. This process is called mounting to find files such as textures, models, (lua) code, maps, mods and more We'll have to make our own such that it will also mount our version of the FAF repository that we downloaded / cloned previously.

Create a new file called 'dev-init.lua' that will represent the contents we want to be mounted in our development environment. As a starting template you can work with:

``` lua 

-- Replace this with the path to your local repository.
repository =  'C:\\Workspace\\forged-alliance-forever-lua'

-- Imports a path file that is written by Forged Alliance Forever right before it starts the game - do not change.
dofile(InitFileDir .. '\\..\\fa_path.lua')

-- The all-mighty path table that will hold all our mounted directories.
path = {}

-- Adds an entry to the path table - ensuring it is properly formatted.
local function mount_dir(dir, mountpoint)
    table.insert(path, { dir = dir, mountpoint = mountpoint } )
end

-- Mounts all folders in the directory to the mount point.
-- Example: Mount all mods at the mount point '/mods/
-- dir        = SHGetFolderPath('PERSONAL') .. 'My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\mods'
-- mountpoint = '/mods'
local function mount_contents(dir, mountpoint)
    LOG('checking ' .. dir)
    for _,entry in io.dir(dir .. '\\*') do
        if entry != '.' and entry != '..' then
            local mp = string.lower(entry)
            mp = string.gsub(mp, '[.]scd$', '')
            mp = string.gsub(mp, '[.]zip$', '')
            mount_dir(dir .. '\\' .. entry, mountpoint .. '/' .. mp)
        end
    end
end

-- The classic Supreme Commander directories in your 'My Documents' folder.
-- They don't work with accents or other foreign characters in usernames.
mount_contents(SHGetFolderPath('PERSONAL') .. 'My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\mods', '/mods')
mount_contents(SHGetFolderPath('PERSONAL') .. 'My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\maps', '/maps')

-- The fall-back vault location. 
-- They don't work with accents or other foreign characters in usernames.
mount_contents(InitFileDir .. '\\..\\user\\My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\mods', '/mods')
mount_contents(InitFileDir .. '\\..\\user\\My Games\\Gas Powered Games\\Supreme Commander Forged Alliance\\maps', '/maps')
mount_**dir**(dev_path, '/')

-- The game data as provided by Steam / GPG.
mount_dir(fa_path .. '\\gamedata\\*.scd', '/')
mount_dir(fa_path, '/')

hook = {
    '/schook'
}

-- Various protocols for communication - unsure what these do.
protocols = {
    'http',
    'https',
    'mailto',
    'ventrilo',
    'teamspeak',
    'daap',
    'im',
}
```

The most important part of this example initialisation file is the following function:

``` lua 
-- Adds an entry to the path table - ensuring it is properly formatted.
local function mount_dir(dir, mountpoint)
    table.insert(path, { dir = dir, mountpoint = mountpoint } )
end
```

It adds a path to mount at some mount point. All the other functions are helper functions - that make mounting entire folders (and their sub folders) easier. For more inspiration you can view the other initialisation files that also include blacklisting and / or whitelisting. The initialisation file used by the came is `init.lua`.

### Shortcut and program arguments

We'll use a shortcut to pass program arguments to the game. Create a shortcut of the executable `ForgedAlliance.exe`. Right click on the shortcut to edit its properties. We'll append information at the end of the target - those are program arguments. As an example we turn:

`C:\ProgramData\FAForever\bin\ForgedAlliance.exe`

Into:

`C:\ProgramData\FAForever\bin\ForgedAlliance.exe /init "init-dev.lua" /showlog /EnableDiskWatch /cheat`

With these program arguments we tell the game that:
 - `/init "init-dev.lua"`: the initialisation file to use, including our workspace.
 - `/showlog`: start with the moholog opened.
 - `/EnableDiskWatch`: actively scan for changed files and reload them live.
 - `/cheat`: enable cheats by default.

### FAQ

#### What is Git?
Git is a widely used version control system for generally, but not exclusively to, source code. It allows people to collaborate quite easily through various features such as branches and merging. As it is beyond the scope of this guide I happily refer you to the following sources:
 - https://guides.github.com/activities/hello-world/
 - https://github.com/about

If you intent to actively develop for the FAF community then I highly recommend you to learn how Git works.

#### What is mounting?
Mounting is the process of telling the game the following information:
 - What folders are interesting to scan for lua, blueprint or other files.
 - How we can refer to those folders in code.

In the initialisation files we have a path variable. This is a table containing the absolute path to a folder along with a relative path or identifier to that folder. An identifier can be shared - if it is used the game will search through all folders with that identifier. As an example of the function that takes care of this for us: 

``` lua 
-- Example input:
-- dir          = C:/Steam/steamapps/common/Supreme Commander/gamedata/env.scd
-- mountpoint   = /env
local function mount_dir(dir, mountpoint)
    table.insert(path, { dir = dir, mountpoint = mountpoint })
end
```

During the initialisation of the game we tell the game all the relevant folders. In the development initialisation we include our version of the FAF repository.

#### About EnableDiskWatch

This program argument tells the game to actively scan for changes in the mounted directories. It will take note of:
 - Changes in your workspace.
 - Changes in map scripts.
 - Changes in mods including blueprint changes, but not script changes.

Blueprint changes are not applied to units that are already active in game. However, the actual blueprint of an already existing unit will contain the new changes. Units that you spawn after the change will properly reflect the adjusted blueprint. tldr; just spawn a new unit.

Scripts of mods are not actively scanned and updated. Luckily, a mod is loaded from scratch when you start a map. Therefore you can just restart the map to fully reload a mod along with its script files.

#### What is a hook?
As this is not neccesarily part of this guide I'll keep it short. A hook is in its core an append of one file to another. Say we have a UI mod that adds UI elements to the game. Generally you'd start a mod like this by hooking the `CreateUI(isReplay)` function in `lua/ui/game/gamemain.lua`. As an example, we have these two files. Take note that these are shortened (with `(...)`) as in: their full content is not shown.

``` lua
-- original file
-- path: /lua/ui/game/gamemain.lua

-- (...)

function CreateUI(isReplay) 
    -- (...)
end

-- (...)
```

``` lua
-- a file part of a mod that hooks safely
-- path: /mods/%MODNAME%/lua/ui/game/gamemain.lua

-- (...)

do 

    local baseCreateUI = CreateUI;

    function CreateUI(isReplay) 
        baseCreateUI(isReplay) 
        -- (...)
    end

end

-- (...)
```

When the mod is mounted the resulting file is similar to:

``` lua 
-- the file that is appended with the hook

-- (...)

function CreateUI(isReplay) 
    -- (...)
end

-- (...)

do 

    local baseCreateUI = CreateUI;

    function CreateUI(isReplay) 
        baseCreateUI(isReplay) 
        -- (...)
    end

end

-- (...)
``` 

The file that hooks is literally appended to the original file. The appended file is then used by the game. The last definition of a function and / or variable is used by the game. A few notes on hooking:
 - A 'safe' hook stores the original function in a separate variable and then calls the original function. If you do not do this then you may override the changes of other mods.
 - A 'safer' hook does all its work inside a `do (...) end` block. This ensures that your local version of the original function is not accidentally overriden by another hook.

#### What is a schook?
Similar to a hook but then within the gamedata folder of the game. These are always added to the game and cannot be selected like mods are.
