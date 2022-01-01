**For Minecraft: Bedrock Edition 1.18.0**

# 4.0.0
Released 1st December 2021.

This major version features substantial changes throughout the core, including significant API changes, new world format support, performance improvements and a network revamp.

Please also note that this changelog is provided on a best-effort basis, and it's possible some changes might not have been mentioned here.
If you find any omissions, please submit pull requests to add them.

* [Core](#core)
  + [General](#general)
  + [Dependency changes](#dependency-changes)
  + [Performance](#performance)
  + [Tools](#tools)
  + [Commands](#commands)
  + [Configuration](#configuration)
  + [World handling](#world-handling)
    - [Interface](#interface)
    - [Functional](#functional)
    - [Performance](#performance-1)
  + [Logging](#logging)
  + [Network](#network)
    - [Performance](#performance-2)
    - [Minecraft Bedrock packet encryption](#minecraft-bedrock-packet-encryption)
    - [Error handling](#error-handling)
    - [New packet handler system](#new-packet-handler-system)
  + [Plugin loading](#plugin-loading)
  + [Internals](#internals)
* [API](#api)
  + [General](#general-1)
  + [Changes to `plugin.yml`](#changes-to-pluginyml)
    - [Permission nesting](#permission-nesting)
    - [`src-namespace-prefix`](#src-namespace-prefix)
  + [Other changes](#other-changes)
  + [Block](#block)
  + [Command](#command)
  + [Entity](#entity)
    - [General](#general-2)
    - [Effect](#effect)
    - [Removal of runtime entity NBT](#removal-of-runtime-entity-nbt)
    - [Entity creation](#entity-creation)
    - [WIP removal of entity network metadata](#wip-removal-of-entity-network-metadata)
  + [Event](#event)
    - [Internal event system no longer depends on `Listener`s](#internal-event-system-no-longer-depends-on-listeners)
    - [Default cancelled handling behaviour has changed](#default-cancelled-handling-behaviour-has-changed)
    - [`PlayerPreLoginEvent` changes](#playerpreloginevent-changes)
    - [Other changes](#other-changes-1)
  + [Inventory](#inventory)
  + [Item](#item)
    - [General](#general-3)
    - [NBT handling](#nbt-handling)
    - [Enchantment](#enchantment)
  + [Lang](#lang)
  + [Network](#network-1)
  + [Permission](#permission)
  + [Player](#player)
  + [Plugin](#plugin)
  + [Promise](#promise)
  + [Scheduler](#scheduler)
    - [Thread-local storage for AsyncTasks](#thread-local-storage-for-asynctasks)
    - [Other AsyncTask changes](#other-asynctask-changes)
    - [Non-AsyncTask changes](#non-asynctask-changes)
  + [Server](#server)
  + [Level / World](#level--world)
    - [General](#general-4)
    - [Particles](#particles)
    - [Sounds](#sounds)
  + [Utils](#utils)
* [Gameplay](#gameplay)
  + [World loading](#world-loading)
  + [Blocks](#blocks)
  + [Items](#items)
  + [Inventory](#inventory-1)
  + [Misc](#misc)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## Core
### General
- Remote console (RCON) has been removed. The [RconServer](https://github.com/pmmp/RconServer) plugin is provided as a substitute.
- Spawn protection has been removed. The [BasicSpawnProtection](https://github.com/pmmp/BasicSpawnProtection) plugin is provided as a substitute.
- Player movement anti-cheat has been removed.
- GS4 Query no longer breaks when disabling RakLib.
- PreProcessor is removed from builds due to high maintenance cost and low benefit. Its usage is now discouraged.
- The title bar no longer displays heap memory usage numbers (nobody seemed to know that was what it was, anyway).
- `start.sh` no longer specifically mentions PHP 7 when erroring because of a missing PHP binary.
- The `--bootstrap` CLI option has been removed.
- Introduced support for connecting via IPv6:
  - PHP binary used must now always be built with IPv6 support, even if IPv6 is disabled. This is because RakNet may still send link-local IPv6 loopback addresses in connection packets even when only using IPv4.
  - Port `19133` is used for IPv6 by default so that Minecraft Bedrock can detect IPv6 servers on LAN.
  - GS4 Query is supported on both IPv4 and IPv6 according to `server.properties` settings.
  - New `server.properties` options `enable-ipv6`, `server-ipv6`, `server-portv6` have been added (see below).

### Dependency changes
- The `pocketmine_chunkutils` PHP extension has been dropped.
- IPv6 support in PHP is now mandatory.
- New PHP extensions are required by this version:
  - [crypto](https://github.com/bukka/php-crypto)
  - [chunkutils2](https://github.com/pmmp/ext-chunkutils2)
  - [gmp](https://www.php.net/manual/en/book.gmp.php)
  - [igbinary](https://github.com/igbinary/igbinary)
  - [leveldb](https://github.com/pmmp/php-leveldb) (must be built with [pmmp/leveldb](https://github.com/pmmp/leveldb/tree/mojang-compatible))
  - [morton](https://github.com/pmmp/ext-morton)
- The following Composer dependency versions have changed (**PLEASE READ, dependency API changes are not mentioned in this changelog!**):
  - `pocketmine/bedrock-protocol` has been added at [7.0.0](https://github.com/pmmp/BedrockProtocol/releases/tag/7.0.0+bedrock-1.18.0).
  - `pocketmine/classloader` has been updated from 0.1.0 to [0.2.0](https://github.com/pmmp/ClassLoader/releases/tag/0.2.0) (**significant API changes, new features**).
  - `pocketmine/color` has been added at [0.2.0](https://github.com/pmmp/Color/releases/tag/0.2.0).
  - `pocketmine/errorhandler` has been added at [0.3.0](https://github.com/pmmp/ErrorHandler/releases/tag/0.3.0).
  - `pocketmine/locale-data` has been added at [2.0.16](https://github.com/pmmp/Language/releases/tag/2.0.16).
  - `pocketmine/log` has been updated from 0.2.0 to [0.4.0](https://github.com/pmmp/Log/releases/tag/0.4.0) (**minor API changes**, see also [0.3.0](https://github.com/pmmp/Log/releases/tag/0.3.0)).
  - `pocketmine/nbt` has been updated from 0.2.18 to [0.3.0](https://github.com/pmmp/NBT/releases/tag/0.3.0) (**significant API changes**).
  - `pocketmine/raklib` has been updated from 0.12.7 to [0.14.2](https://github.com/pmmp/RakLib/releases/tag/0.14.2) (**significant API changes**, see also [0.13.0](https://github.com/pmmp/RakLib/releases/tag/0.13.0)).
  - `pocketmine/raklib-ipc` has been added at [0.1.0](https://github.com/pmmp/RakLibIpc/releases/tag/0.1.0) (components extracted from RakLib).
  - `pocketmine/snooze` has been updated from 0.1.0 to [0.3.0](https://github.com/pmmp/Snooze/releases/tag/0.3.0) (**minor API changes**, see also [0.2.0](https://github.com/pmmp/Snooze/releases/tag/0.2.0)).
  - `pocketmine/spl` has been dropped.
- Minecraft Bedrock protocol is now developed in a separate repository, [pmmp/BedrockProtocol](https://github.com/pmmp/BedrockProtocol).
  - It has significant changes compared to PM3. Please read its changelogs.
- The following submodules have been removed:
  - `resources/vanilla`: BedrockData is now included via Composer dependency [`pocketmine/bedrock-data`](https://packagist.org/packages/pocketmine/bedrock-data).
  - `resources/locale`: Language files are now included via Composer dependency [`pocketmine/locale-data`](https://packagist.org/packages/pocketmine/locale-data).
  - This means it's now possible to run a server from git sources without cloning submodules :)
  - All remaining submodules (DevTools, build/php) are non-essential for building and running a server.

### Performance
- `/op`, `/deop`, `/whitelist add` and `/whitelist remove` no longer cause player data to be loaded from disk for no reason.
- Timings now use high-resolution timers provided by `hrtime()` to collect more accurate performance metrics.
- Closures are now used for internal event handler calls. This provides a performance improvement of 10-20% over the 3.x system, which had to dynamically resolve callables for every event call.
- Improved startup performance when loading many plugins.
- See more in the [Worlds / Performance](#performance-2) and [Network / Performance](#performance-3) sections.

### Tools
Some new scripts have been added in the `tools/` directory of the repository. These scripts may use the PocketMine-MP core library, but are intended to be run standalone.

 - `convert-world.php`: allows converting a world to a new format without a running server
 - `compact-regions.php`: repacks files in legacy Region worlds to clean up wasted disk space
 - `generate-permission-doc.php`: generates a Markdown document of all core permissions (see [example](https://gist.github.com/dktapps/eed6d6a4571f01b676236bf9ff2779b2))
 - `simulate-chunk-selector.php`: generates a series of images to visualize the behaviour of the chunk sending algorithm; these images can then be stitched into video using a tool such as [ffmpeg](https://ffmpeg.org/)

### Commands
- The `/effect` command no longer supports numeric IDs - it's now required to use names.
- The `/enchant` command no longer supports numeric IDs - it's now required to use names.
- The `/give` command no longer permits giving items with invalid NBT (e.g. incorrect types). Previously, this was the cause of random server crashes when using items on PM3.
- The `/give` command now supports many new aliases like in Java, e.g. it's now possible to do `/give someone bonemeal` or `/give someone lapis_lazuli` instead of using legacy id:metadata.
- The `/help` command is now localized according to language set in `server.properties`.
- The `/reload` command has been removed.
- The `/setworldspawn` command now accepts relative coordinates when used as a player.
- The `/status` command no longer displays heap memory usage numbers.
- Added `/clear` command with functionality equivalent to that of vanilla Minecraft.
- The following commands' outputs are now localized according to the chosen language settings:
  - `/gc`
  - `/status`
  - `/op`
  - `/deop`
- Fixed use of commands without the proper permission sending a message `commands.generic.permission` instead of the proper message.
- Fixed commands not working in some cases after using some control sequences on the console.
- Fixed `/setworldspawn` setting incorrect positions based on player position when in negative coordinates.

### Configuration
- World presets can now be provided as a `preset` key in `pocketmine.yml`, instead of putting them in the `generator` key.
- The `worlds` config no longer supports attaching the generator settings to the `generator` key (use the `preset` key instead).
- Using an unknown generator in `server.properties` or `pocketmine.yml` will now cause a failure to generate the world.
- Using invalid/incorrect world generator options (presets) in `server.properties` or `pocketmine.yml` will now cause a failure to generate the world.
- The following new options have been added to `pocketmine.yml`:
  - `chunk-ticking.blocks-per-subchunk-per-tick` (default `3`): Increasing this value will increase the rate at which random block updates happen (e.g. grass growth).
  - `network.enable-encryption` (default `true`): Controls whether Minecraft network packets are encrypted or not.
- The following options have been removed from `pocketmine.yml`:
  - `chunk-ticking.light-updates`: Since lighting is needed for basic vanilla functionality to work, allowing this to be disabled without disabling chunk ticking made no sense. If you don't want light calculation to occur, you can disable chunk ticking altogether by setting `chunk-ticking.per-tick` to `0` in `pocketmine.yml`.
  - `player.anti-cheat.allow-movement-cheats`
- The following new options have been added to `server.properties`:
  - `enable-ipv6`: `on` by default. Disabling this completely disables IPv6 support.
  - `server-ipv6`: `::` by default (equivalent to "any IP", like `0.0.0.0` for IPv4). Most users shouldn't need to change this setting, and it doesn't appear in `server.properties` by default.
  - `server-portv6`: `19133` by default. You may run both IPv4 and IPv6 on the same port, but since Bedrock scans on 19133 by default, PM also uses the same.

### World handling
#### Interface
- Progress of spawn terrain chunk generation is now logged during initial world creation.

#### Functional
- Minecraft Bedrock worlds up to 1.12.x are now supported. (1.13+ are still **not supported** due to yet another format change, which is large and requires a lot of work).
- Automatic conversion of deprecated world formats is now implemented.
- All formats except `leveldb` have been deprecated. The following world formats will be **automatically converted on load to a new format**:
  - `mcregion`
  - `anvil`
  - `pmanvil`
- Generator options of existing worlds are now validated before loading them. If they are invalid, the server will fail to load them.
- Fixed the server attempting to generate a world if it failed to load.
- 256 build-height is now supported in all worlds (facilitated by automatic conversion).
- Extended blocks are now supported (facilitated by automatic conversion).
- The server will now attempt to translate invalid blocks to valid equivalents when loading chunks. This fixes many issues with `update!` blocks appearing in worlds, particularly ghost structures (these would appear when world editors previously erased some blocks by setting their IDs but not metadata).
- Lighting is no longer stored or loaded from disk - instead, it's calculated on the fly as-needed. This fixes many long-standing bugs.
- Explosions now use the standard mechanism for processing block updates. Previously, it used a special mechanism due to prohibitively poor efficiency of the standard algorithm. Since these inefficiencies have now been addressed, explosions can now be consistent with everything else, with minimal performance impact.
- Fixed debug spam of `chunk has no loaders registered` messages during chunk generation.
- Various cases of corrupted data in chunks will now cause an error to be logged instead of a server crash. This includes tiles with colliding positions and tiles in incorrect, non-loaded chunks.
- Fixed players re-requesting the same ungenerated chunks multiple times before they were sent.
- Fixed players re-requesting chunks when turning their heads or jumping.

#### Performance
- `leveldb` is now the primary supported world format. It is inherently faster than region-based formats thanks to better design.
- Partial chunk saves (only saving modified subcomponents of chunks) has been implemented. This drastically reduces the amount of data that is usually necessary to write on chunk save, which in turn **drastically reduces the time to complete world saves**. This is possible thanks to the modular design of the `leveldb` world format - this enhancement is not possible with region-based formats.
- Lighting is no longer guaranteed to be available on every chunk. It's now calculated on the fly as-needed.
- Z-order curves (morton codes) are now used for block and chunk coordinate hashes. This substantially improves performance in many areas by resolving a hashtable key hash collision performance issue. Affected areas include explosions, light calculation, and more.
- Improved performance of `World->setBlock()` by around 35% when the `$update` parameter is set to `true`.
- Improved performance of `World->setBlock()` by around 30% when the `$update` parameter is set to `false`.

### Logging
- Many components now have a dedicated logger which automatically adds [prefixes] to their messages.
- Main logger now includes milliseconds in timestamps.
- Debug messages are now logged when reach distance checks prevent players from doing something.
- Various messages related to world loading/generation/conversion and plugin loading errors are now localized according to the language set in `server.properties`.
- Exception log format has been changed. Now, exception info is logged in one big block message. This saves space on the console and improves readability, as well as reducing the ability for bad `ThreadedLoggerAttachment`s to break exception output.
- Improved error messages when a plugin command name or alias contains an illegal character.
- The server will now log an EMERGENCY-level message when `Server->forceShutdown()` is used for any other reason than a graceful shutdown.

### Network
This version features substantial changes to the network system, improving coherency, reliability and modularity.

#### Performance
- [`libdeflate`](https://github.com/ebiggers/libdeflate) is now (optionally) used for outbound Minecraft packet compression. It's more than twice as fast as zlib in most cases, providing significant performance boosts to packet broadcasts and overall network performance.

#### Minecraft Bedrock packet encryption
- This fixes replay attacks where hackers steal and replay player logins.
- A new setting has been added to `pocketmine.yml`: `network.enable-encryption` which is `true` by default.

#### Error handling
- Only `BadPacketException` is now caught during packet decode and handling. This requires that all decoding MUST perform proper data error checking.
  - Throwing a `BadPacketException` from decoding will now cause players to be kicked with the message `Packet processing error`.
  - The disconnect message includes a random hex ID to help server owners identify the problems reported by their players.
- Throwing any other exception will now cause a server crash. `Internal server error` has been removed.
- It is now illegal to send a clientbound packet to the server. Doing so will result in the client being kicked with the message `Unexpected non-serverbound packet`.
- Fixed server crash when unable to bind to the desired port. Now, the server will show an error and gracefully stop without a crashdump instead.

#### New packet handler system
- Packet handlers have been separated from NetworkSession into a dedicated packet handler structure.
- A network session may have exactly 1 handler at a time, which is mutable and may be replaced at any time. This allows packet handling logic to be broken up into multiple stages:
  - preventing undefined behaviour when sending wrong packets at the wrong time (they'll now be silently dropped)
  - allowing the existence of ephemeral state-specific logic (for example stricter resource packs download checks)
- Packet handlers are now almost entirely absent from `Player` and instead appear in their own dedicated units.
- Almost all game logic that was previously locked up inside packet handlers in `Player` has been extracted into new API methods. See Player API changes for details.

### Plugin loading
- Phar plugins are now able to depend on folder plugins loaded by DevTools.
- A new "plugin greylist" feature has been introduced, which allows whitelisting or blacklisting plugins from loading. See `plugin_list.yml`.

### Internals
- The `pocketmine` subdirectory has been removed from `src`. [PSR-4 autoloading is now used thanks to Composer](https://github.com/pmmp/PocketMine-MP/blob/4.0.0/composer.json#L63).
- Crashdump rendering has been separated from crashdump data generation. This allows rendering crashdumps from existing JSON data.
- Direct iteration of arrays with string keys is now disallowed by a custom PHPStan rule. This is because numeric strings are casted to integers when used as array keys, which produces a variety of unexpected behaviour particularly for iteration.
  - To iterate on arrays with string keys, `Utils::stringifyKeys()` must now be used.
- Fixed various crashes involving arrays with numeric string keys.

## API
### General
- Most places which previously allowed `callable` now only allow `\Closure`. This is because closures have more consistent behaviour and are more performant.
- `void` and `?nullable` parameter and return types have been applied in many places.
- Everything in the `pocketmine\metadata` namespace and related implementations have been removed.

### Changes to `plugin.yml`
#### Permission nesting
Permission nesting is no longer supported in `plugin.yml`. Grouping permissions (with defaults) in `plugin.yml` had very confusing and inconsistent behaviour.
Instead of nesting permission declarations, they should each be declared separately.

_Before_:
```
permissions:
   pmmp:
     default: op
	 children:
	     pmmp.something:
		     default: op
         pmmp.somethingElse
		     default: op
```

_After_:
```
permissions:
    pmmp.something:
        default: op
    pmmp.somethingElse
        default: op
```

#### `src-namespace-prefix`
A new directive `src-namespace-prefix` has been introduced. This allows you to get rid of those useless subdirectories in a plugin's structure.
For example, a plugin whose main was `pmmp\TesterPlugin\Main` used to have to be structured like this:
```
|-- plugin.yml
|-- src/
    |-- pmmp/
        |-- TesterPlugin/
            |-- Main.php
            |-- SomeOtherClass.php
            |-- SomeNamespace/
                |-- SomeNamespacedClass.php
```
However, if we add `src-namespace-prefix: pmmp\TesterPlugin` to the `plugin.yml`, now we can get rid of the useless directories and structure it like this instead:
```
|-- plugin.yml
|-- src/
    |-- Main.php
    |-- SomeOtherClass.php
    |-- SomeNamespace/
        |-- SomeNamespacedClass.php
```

**Note**: The old structure will also still work just fine. This is not a required change.

### Other changes
- Incorrect structure of `commands` is now detected earlier and handled more gracefully.
- Commands must now declare `permission` for each command. Previously, the `permission` key was optional, causing anyone to be able to use the command.
  - This behaviour was removed because of the potential for security issues - a typo in `plugin.yml` could lead to dangerous functionality being exposed to everyone.
  - If you want to make a command that everyone can use, declare a permission with a `default` of `true` and assign it to the command.
- Permissions must now declare `default` for each permission. Previously, the `default` key was optional, causing the permission to silently be denied to everyone (PM4) or granted to ops implicitly (PM3).

### Block
- A new `VanillaBlocks` class has been added, which contains static methods for creating any currently-known block type. This should be preferred instead of use of `BlockFactory::get()` where constants were used.
- `BlockFactory` is now a singleton, and its methods are no longer static. `BlockFactory::whatever()` should be replaced with `BlockFactory::getInstance()->whatever()`.
- `BlockFactory->get()` is now **deprecated**.
  - For most cases, `VanillaBlocks::WHATEVER_BLOCK()` should fill your needs.
  - `BlockFactory` should now only be used for loading old save data from, for example, a database, config or a world save.
  - To refer to blocks by name, consider using `StringToItemParser` to accept names instead of IDs.
- Blocks now contain their positions instead of extending `Position`. `Block->getPosition()` has been added.
- Blocks with IDs >= 256 are now supported.
- Block state and variant metadata have been separated.
  - Variant is considered an extension of ID and is immutable.
  - `Block->setDamage()` has been removed.
- All blocks now have getters and setters for their appropriate block properties, such as facing, lit/unlit, colour (in some cases), and many more. These should be used instead of metadata.
- Tile entities are now created and deleted automatically when `World->setBlock()` is used with a block that requires a tile entity.
- Some tile entities' API has been exposed on their corresponding `Block` classes, with the tile entity classes being deprecated.
- The `pocketmine\tile` namespace has been relocated to `pocketmine\block\tile`.
- `Block->recalculateBoundingBox()` and `Block->recalculateCollisionBoxes()` are now expected to return AABBs relative to `0,0,0` instead of their own position.
- Block break-info has been extracted into a new dynamic `BlockBreakInfo` unit. The following methods have been moved:
  - `Block->getBlastResistance()` -> `BlockBreakInfo->getBlastResistance()`
  - `Block->getBreakTime()` -> `BlockBreakInfo->getBreakTime()`
  - `Block->getHardness()` -> `BlockBreakInfo->getHardness()`
  - `Block->getToolHarvestLevel()` -> `BlockBreakInfo->getToolHarvestLevel()`
  - `Block->getToolType()` -> `BlockBreakInfo->getToolType()`
  - `Block->isBreakable()` -> `BlockBreakInfo->isBreakable()`
  - `Block->isCompatibleWithTool()` -> `BlockBreakInfo->isToolCompatible()`
- The following API methods have been added:
  - `Block->asItem()`: returns an itemstack corresponding to the block
  - `Block->getModelPositionOffset()`: used to offset the bounding box of blocks like bamboo based on coordinates
  - `Block->isSameState()`: returns whether the block is the same as the parameter, including state information
  - `Block->isSameType()`: returns whether the block is the same as the parameter, without state information
  - `Block->isFullCube()`
  - `Liquid->getMinAdjacentSourcesToFormSource()`: returns how many adjacent source blocks of the same liquid must be present in order for the current block to become a source itself
- The following hooks have been added:
  - `Block->onAttack()`: called when a player in survival left-clicks the block to try to start breaking it
  - `Block->onEntityLand()`: called when an entity lands on this block after falling (from any distance)
  - `Block->onPostPlace()`: called directly after placement in the world, handles things like rail connections and chest pairing
- The following API methods have been renamed:
  - `Block->getDamage()` -> `Block->getMeta()`
  - `Block->onActivate()` -> `Block->onInteract()`
  - `Block->onEntityCollide()` -> `Block->onEntityInside()`
- The following API methods have changed signatures:
  - `Block->onInteract()` now has the signature `onInteract(Item $item, int $face, Vector3 $clickVector, ?Player $player = null) : bool`
  - `Block->getCollisionBoxes()` is now final. Classes should override `recalculateCollisionBoxes()`.
- The following API methods have been removed:
  - `Block->canPassThrough()`
  - `Block->setDamage()`
  - `Block::get()`: this was superseded by `BlockFactory::get()` a long time ago
  - `Block->getBoundingBox()`
- The following classes have been added:
  - `inventory\CraftingTableInventory`: represents a crafting table's 3x3 crafting grid
  - `utils\LeverFacing`
  - `utils\MinimumFlowCostCalculator`: encapsulates flow calculation logic previously locked inside `Liquid`.
- The following classes have been renamed:
  - `BlockIds` -> `BlockLegacyIds`
  - `CobblestoneWall` -> `Wall`
  - `NoteBlock` -> `Note`
  - `SignPost` -> `FloorSign`
  - `StandingBanner` -> `FloorBanner`
- The following classes have been removed:
  - `Bricks`
  - `BurningFurnace`
  - `CobblestoneStairs`
  - `Dandelion`
  - `DoubleSlab`
  - `DoubleStoneSlab`
  - `EndStone`
  - `GlowingRedstoneOre`
  - `GoldOre`
  - `Gold`
  - `IronDoor`
  - `IronOre`
  - `IronTrapdoor`
  - `Iron`
  - `Lapis`
  - `NetherBrickFence`
  - `NetherBrickStairs`
  - `Obsidian`
  - `PurpurStairs`
  - `Purpur`
  - `QuartzStairs`
  - `Quartz`
  - `RedSandstoneStairs`
  - `RedSandstone`
  - `SandstoneStairs`
  - `Sandstone`
  - `StainedClay`
  - `StainedGlassPane`
  - `StainedGlass`
  - `StoneBrickStairs`
  - `StoneBricks`
  - `StoneSlab2`
  - `StoneSlab`
  - `Stone`
  - `WallBanner`
  - `WallSign`
  - `Wood2`
- `BlockToolType` constants have been renamed to remove the `TYPE_` prefix.

### Command
- The following classes have been removed:
  - `RemoteConsoleCommandSender`
- The following API methods have signature changes:
  - `Command->setPermission()` argument is now mandatory (but still nullable).
  - `CommandSender->setScreenLineHeight()` argument is now mandatory (but still nullable).
  - `Command->getDescription()` now returns `Translatable|string`.
  - `Command->getUsage()` now returns `Translatable|string`.
  - `Command->setDescription()` now accepts `Translatable|string`.
  - `Command->setUsage()` now accepts `Translatable|string`.
- `Command->setPermission()` now throws an exception if given a string containing non-existing permissions. Previously, it would silently default to allowing ops to use the command, which may not have been desired.
  - This is usually caused by a typo or forgotten permission declaration.
- Commands with spaces in the name are no longer supported.
- Command usage strings and description strings are no longer automatically translated (use `Translatable` instead of bare string keys).

### Entity
#### General
- `Entity` no longer extends from `Location`. `Entity->getLocation()` and `Entity->getPosition()` should be used instead.
- Ender inventory has been refactored. It's now split into two parts:
  - `EnderChestInventory` is a temporary gateway "inventory" that acts as a proxy to the player's ender inventory. It has a `Position` for holder. This is discarded when the player closes the inventory window.
  - `PlayerEnderInventory` is the storage part. This is stored in `Human` and does not contain any block info.
  - To open the player's ender inventory, use `Player->setCurrentWindow(new EnderChestInventory($blockPos, $player->getEnderInventory()))`.
- The following public fields have been removed:
  - `Entity->chunk`: Entities no longer know which chunk they are in (the `World` now manages this instead).
  - `Entity->height`: moved to `EntitySizeInfo`; use `Entity->size` instead
  - `Entity->width`: moved to `EntitySizeInfo`; use `Entity->size` instead
  - `Entity->eyeHeight`: moved to `EntitySizeInfo`; use `Entity->size` instead
- The following API methods have been added:
  - `Entity->getFallDistance()`
  - `Entity->setFallDistance()`
  - `ItemEntity->getDespawnDelay()`
  - `ItemEntity->setDespawnDelay()`
  - `Living->calculateFallDamage()`: this is `protected`, and may be overridden by subclasses to provide custom damage logic
  - `Human->getHungerManager()`
  - `Human->getXpManager()`
- The following methods have signature changes:
  - `Entity->entityBaseTick()` is now `protected`.
  - `Entity->move()` is now `protected`.
  - `Entity->setPosition()` is now `protected` (use `Entity->teleport()` instead).
  - `Entity->setPositionAndRotation()` is now `protected` (use `Entity->teleport()` instead).
  - `Living->knockBack()` now accepts `float, float, float` (the first two parameters have been removed).
  - `Living->getEffects()` now returns `EffectManager` instead of `Effect[]`.
  - `Location->__construct()` now accepts `?World $world` in the 4th parameter, and all parameters are now mandatory.
- The following classes have been added:
  - `effect\EffectManager`: contains effect-management functionality extracted from `Living`
  - `HungerManager`: contains hunger-management functionality extracted from `Human`
  - `ExperienceManager`: contains XP-management functionality extracted from `Human`
- The following API methods have been moved / renamed:
  - `Entity->fall()` -> `Entity->onHitGround()` (and visibility changed to `protected` from `public`)
  - `Living->removeAllEffects()` -> `EffectManager->clear()`
  - `Living->removeEffect()` -> `EffectManager->remove()`
  - `Living->addEffect()` -> `EffectManager->add()`
  - `Living->getEffect()` -> `EffectManager->get()`
  - `Living->hasEffect()` -> `EffectManager->has()`
  - `Living->hasEffects()` -> `EffectManager->hasEffects()`
  - `Living->getEffects()` -> `EffectManager->all()`
  - `Human->getFood()` -> `HungerManager->getFood()`
  - `Human->setFood()` -> `HungerManager->setFood()`
  - `Human->getMaxFood()` -> `HungerManager->getMaxFood()`
  - `Human->addFood()` -> `HungerManager->addFood()`
  - `Human->isHungry()` -> `HungerManager->isHungry()`
  - `Human->getEnderChestInventory()` -> `Human->getEnderInventory()`
  - `Human->getSaturation()` -> `HungerManager->getSaturation()`
  - `Human->setSaturation()` -> `HungerManager->setSaturation()`
  - `Human->addSaturation()` -> `HungerManager->addSaturation()`
  - `Human->getExhaustion()` -> `HungerManager->getExhaustion()`
  - `Human->setExhaustion()` -> `HungerManager->setExhaustion()`
  - `Human->exhaust()` -> `HungerManager->exhaust()`
  - `Human->getXpLevel()` -> `ExperienceManager->getXpLevel()`
  - `Human->setXpLevel()` -> `ExperienceManager->setXpLevel()`
  - `Human->addXpLevels()` -> `ExperienceManager->addXpLevels()`
  - `Human->subtractXpLevels()` -> `ExperienceManager->subtractXpLevels()`
  - `Human->getXpProgress()` -> `ExperienceManager->getXpProgress()`
  - `Human->setXpProgress()` -> `ExperienceManager->setXpProgress()`
  - `Human->getRemainderXp()` -> `ExperienceManager->getRemainderXp()`
  - `Human->getCurrentTotalXp()` -> `ExperienceManager->getCurrentTotalXp()`
  - `Human->setCurrentTotalXp()` -> `ExperienceManager->setCurrentTotalXp()`
  - `Human->addXp()` -> `ExperienceManager->addXp()`
  - `Human->subtractXp()` -> `ExperienceManager->subtractXp()`
  - `Human->getLifetimeTotalXp()` -> `ExperienceManager->getLifetimeTotalXp()`
  - `Human->setLifetimeTotalXp()` -> `ExperienceManager->setLifetimeTotalXp()`
  - `Human->canPickupXp()` -> `ExperienceManager->canPickupXp()`
  - `Human->onPickupXp()` -> `ExperienceManager->onPickupXp()`
  - `Human->resetXpCooldown()` -> `ExperienceManager->resetXpCooldown()`
- The following API methods have been removed:
  - `Human->getRawUniqueId()`: use `Human->getUniqueId()->toBinary()` instead
- The following classes have been removed:
  - `Creature`
  - `Damageable`
  - `Monster`
  - `NPC`
  - `Rideable`
  - `Vehicle`
- `Skin` now throws exceptions on creation if given invalid data.
- Fixed `Living->lookAt()` not taking eye height into account.

#### Effect
- All `Effect` related classes have been moved to the `pocketmine\entity\effect` namespace.
- Effect functionality embedded in the `Effect` class has been separated out into several classes. The following classes have been added:
  - `AbsorptionEffect`
  - `HealthBoostEffect`
  - `HungerEffect`
  - `InstantDamageEffect`
  - `InstantEffect`
  - `InstantHealthEffect`
  - `InvisibilityEffect`
  - `LevitationEffect`
  - `PoisonEffect`
  - `RegenerationEffect`
  - `SaturationEffect`
  - `SlownessEffect`
  - `SpeedEffect`
  - `WitherEffect`
- `VanillaEffects` class has been added. This exposes all vanilla effect types as static methods, replacing the old `Effect::getEffect()` nastiness.
  - Example: `Effect::getEffect(Effect::NIGHT_VISION)` can be replaced by `VanillaEffects::NIGHT_VISION()`.
- Negative effect amplifiers are now explicitly disallowed due to undefined behaviour they created.
- The boundaries between MCPE effect IDs and PocketMine-MP internals are now more clear.
  - ID handling is moved to `pocketmine\data\bedrock\EffectIdMap`.
  - All effect ID constants have been removed from `Effect`. `pocketmine\data\bedrock\EffectIds` if you still need legacy effect IDs for some reason.
- The following API methods have been moved:
  - `Effect->getId()` -> `EffectIdMap->toId()`
  - `Effect::registerEffect()` -> `EffectIdMap->register()`
  - `Effect::getEffect()` -> `EffectIdMap->fromId()`
  - `Effect::getEffectByName()` -> `StringToEffectParser->parse()`
- Added `StringToEffectParser` singleton:
  - Supports custom aliases!
  - This is used by `/effect` to provide name support.

#### Removal of runtime entity NBT
- Entities no longer keep their NBT alive at runtime.
  - `Entity->namedtag` has been removed.
  - `Entity->saveNBT()` now returns a newly created `CompoundTag` instead of modifying the previous one in-place.
  - `Entity->initEntity()` now accepts a `CompoundTag` parameter.

#### Entity creation
- `Entity::createEntity()` has been removed. It's no longer needed for creating new entities at runtime - just use `new YourEntity` instead.
- `Entity` subclass constructors can now have any signature, just like a normal class.
- Loading entities from NBT is now handled by `EntityFactory`. It works quite a bit differently than `Entity::createEntity()` did. Instead of registering `YourEntity::class` to a set of Minecraft save IDs, you now need to provide a callback which will construct an entity when given some NBT and a `World`.
  - `EntityFactory` is a singleton. You can get its instance by using `EntityFactory::getInstance()`.
  - Creation callbacks are registered using `EntityFactory->register()`.
  - Creation callbacks must have the signature `function(World, CompoundTag) : Entity`.
  - This enables `Entity` subclasses to have any constructor parameters they like.
  - It also allows requiring that certain data is always provided (for example, it doesn't make much sense to create a `FallingBlock` without specifying what type of block).
  - Examples:
    - `ItemEntity` now requires an `Item` in its constructor, so its creation callback decodes the `Item` from the NBT to be passed to the constructor.
    - `Painting` now requires a `PaintingMotive` in its constructor, so its creation callback decides which `PaintingMotive` to provide based on the NBT it receives.
    - See `EntityFactory` for more examples.
- `EntityFactory->register()` (previously `Entity::registerEntity()`) will now throw exceptions on error cases instead of returning `false`.
- The following API methods have been moved:
  - `Entity::registerEntity()` -> `EntityFactory->register()`
- The following classes have changed constructors:
  - All projectile subclasses now require a `?Entity $thrower` parameter.
  - `Arrow->__construct()` now requires a `bool $critical` parameter (in addition to the `$thrower` parameter).
  - `ExperienceOrb->__construct()` now requires a `int $xpValue` parameter.
  - `FallingBlock->__construct()` now requires a `Block $block` parameter.
  - `ItemEntity->__construct()` now requires an `Item $item` parameter.
  - `Painting->__construct()` now requires a `PaintingMotive $motive` parameter.
  - `SplashPotion->__construct()` now requires a `int $potionId` parameter.
- The following API methods have been removed:
  - `Entity::createBaseNBT()`: `new YourEntity` and appropriate API methods should be used instead
  - `Entity->getSaveId()`
  - `Entity::getKnownEntityTypes()`
  - `Entity::createEntity()`: use `new YourEntity` instead (to be reviewed)

#### WIP removal of entity network metadata
- All network metadata related constants have been removed from the `Entity` class and moved to the protocol layer. It is intended to remove network metadata from the API entirely, but this has not yet been completed.
  - `Entity::DATA_FLAG_*` constants have been moved to `pocketmine\network\mcpe\protocol\types\entity\EntityMetadataFlags`.
  - `Entity::DATA_TYPE_*` constants have been moved to `pocketmine\network\mcpe\protocol\types\entity\EntityMetadataTypes`.
  - `Entity::DATA_*` constants have been moved to `pocketmine\network\mcpe\protocol\types\entity\EntityMetadataProperties`.
- `DataPropertyManager` has been moved to the `pocketmine\network\mcpe\protocol\types\entity` namespace, and as such isn't considered part of the API anymore.
- Introduced internal `Entity` hook `syncNetworkData()`. This function is expected to synchronize entity properties with the entity's network data set.
- Internal usage of network metadata sets to store internal entity properties has been removed. Entities are now expected to use regular class properties and synchronize with the network data set as-asked.
- `Entity->propertyManager` has been renamed to `Entity->networkProperties`.
- `Entity->getDataPropertyManager()` has been renamed to `Entity->getNetworkProperties()`.

### Event
#### Internal event system no longer depends on `Listener`s
- The internal event processing system no longer depends on `Listener` objects. Arbitrary closures can now be used, provided that they satisfy the standard requirements to be a handler.
  - This change improves performance of event handler calling by approximately 15%. This does not include anything plugins are doing.
  - The following classes have been removed:
    - `pocketmine\plugin\EventExecutor`
    - `pocketmine\plugin\MethodEventExecutor`
  - `RegisteredListener->__construct()` now requires `Closure` instead of `Listener, EventExecutor` as the leading parameters.
  - `RegisteredListener->getListener()` has been removed.

#### Default cancelled handling behaviour has changed
- Handler functions will now **not receive cancelled events by default**. This is a **silent BC break**, i.e. it won't raise errors, but it might cause bugs.
- `@ignoreCancelled` is now no longer respected.
- `@handleCancelled` has been added. This allows opting _into_ receiving cancelled events (it's the opposite of `@ignoreCancelled`).
  - `@handleCancelled` may not be used on non-cancellable events (an exception will be thrown during registration).

#### `PlayerPreLoginEvent` changes
- The `Player` object no longer exists at this phase of the login. Instead, a `PlayerInfo` object is provided, along with connection information.
- Ban, server-full and whitelist checks are now centralized to `PlayerPreLoginEvent`. It's no longer necessary (or possible) to intercept `PlayerKickEvent` to handle these types of disconnects.
  - Multiple kick reasons may be set to ensure that the player is still removed if there are other reasons for them to be disconnected and one of them is cleared. For example, if a player is banned and the server is full, clearing the ban flag will still cause the player to be disconnected because the server is full.
  - Plugins may set custom kick reasons. Any custom reason has absolute priority.
  - If multiple flags are set, the kick message corresponding to the highest priority reason will be shown. The priority (as of this snapshot) is as follows:
    - Custom (highest priority)
    - Server full
    - Whitelisted
    - Banned
  - The `PlayerPreLoginEvent::KICK_REASON_PRIORITY` constant contains a list of kick reason priorities, highest first.
- The following constants have been added:
  - `PlayerPreLoginEvent::KICK_REASON_PLUGIN`
  - `PlayerPreLoginEvent::KICK_REASON_SERVER_FULL`
  - `PlayerPreLoginEvent::KICK_REASON_SERVER_WHITELISTED`
  - `PlayerPreLoginEvent::KICK_REASON_BANNED`
  - `PlayerPreLoginEvent::KICK_REASON_PRIORITY`: ordered list of kick reason priorities, highest first
- The following API methods have been added:
  - `PlayerPreLoginEvent->clearAllKickReasons()`
  - `PlayerPreLoginEvent->clearKickReason()`
  - `PlayerPreLoginEvent->getFinalKickMessage()`: the message to be shown to the player with the current reason list in place
  - `PlayerPreLoginEvent->getIp()`
  - `PlayerPreLoginEvent->getKickReasons()`: returns an array of flags indicating kick reasons, must be empty to allow joining
  - `PlayerPreLoginEvent->getPlayerInfo()`
  - `PlayerPreLoginEvent->getPort()`
  - `PlayerPreLoginEvent->isAllowed()`
  - `PlayerPreLoginEvent->isAuthRequired()`: whether XBL authentication will be enforced
  - `PlayerPreLoginEvent->isKickReasonSet()`
  - `PlayerPreLoginEvent->setAuthRequired()`
  - `PlayerPreLoginEvent->setKickReason()`
- The following API methods have been changed:
  - `PlayerPreLoginEvent->getKickMessage()` now has the signature `getKickMessage(int $flag) : ?string`
- The following API methods have been removed:
  - `PlayerPreLoginEvent->setKickMessage()`
  - `PlayerPreLoginEvent->getPlayer()`
- The following API methods have been moved / renamed:
  - `InventoryPickupItemEvent->getItem()` -> `InventoryPickupItemEvent->getItemEntity()`

#### Other changes
- Disconnecting players during events no longer crashes the server (although it might cause other side effects).
- Cancellable events must now implement `CancellableTrait` to get the cancellable components needed to satisfy interface requirements. `Event` no longer stubs these methods.
- `PlayerInteractEvent` is no longer fired when a player activates an item. This fixes the age-old complaint of `PlayerInteractEvent` firing multiple times when interacting once. The following constants have been removed:
  - `PlayerInteractEvent::LEFT_CLICK_AIR`
  - `PlayerInteractEvent::RIGHT_CLICK_AIR`
  - `PlayerInteractEvent::PHYSICAL`
- The following events have been added:
  - `BlockTeleportEvent`: block teleporting, for example dragon egg when attacked.
  - `EntityItemPickupEvent`: player (or other entity) picks up a dropped item (or arrow). Replaces `InventoryPickupItemEvent` and `InventoryPickupArrowEvent`.
    - Unlike its predecessors, this event supports changing the destination inventory.
    - If the destination inventory is `null`, the item will be destroyed. This is usually seen for creative players with full inventories.
  - `EntityTrampleFarmlandEvent`: mob (or player) jumping on farmland causing it to turn to dirt
  - `PlayerDisplayNameChangeEvent`
  - `PlayerEmoteEvent`
  - `PlayerEntityInteractEvent`: player right-clicking (or long-clicking on mobile) on an entity.
  - `PlayerItemUseEvent`: player activating their held item, for example to throw it.
  - `StructureGrowEvent`: called when trees or bamboo grow (or any other multi-block plant structure).
- The following events have changed behaviour:
  - Bone meal is now consistently never consumed when `BlockGrowEvent` or `StructureGrowEvent` is cancelled.
  - `BlockGrowEvent` is now called when cocoa pods grow.
  - `ChunkPopulateEvent` is now called after all adjacent chunks modified during population have been updated. This fixes issues with modifications made in the event sometimes disappearing.
  - `InventoryOpenEvent` is now fired when a player opens a crafting table's UI.
  - `InventoryCloseEvent` is now fired when a player closes a crafting table's UI.
  - `PlayerDropItemEvent` will now prevent the drops from force-closing of the following inventories:
    - anvil
    - enchanting table
    - loom
  - `PlayerKickEvent` is no longer fired for disconnects that occur before the player completes the initial login sequence (i.e. completing downloading resource packs).
- The following events have been removed:
  - `EntityArmorChangeEvent`
  - `EntityInventoryChangeEvent`
  - `EntityLevelChangeEvent` - `EntityTeleportEvent` with world checks should be used instead.
  - `InventoryPickupArrowEvent` - use `EntityItemPickupEvent` instead
  - `InventoryPickupItemEvent` - use `EntityItemPickupEvent` instead
  - `NetworkInterfaceCrashEvent`
  - `PlayerCheatEvent`
  - `PlayerIllegalMoveEvent`
- The following API methods have been added:
  - `EntityDeathEvent->getXpDropAmount()`
  - `EntityDeathEvent->setXpDropAmount()`
  - `PlayerDeathEvent->getXpDropAmount()`
  - `PlayerDeathEvent->setXpDropAmount()`
- The following API methods have been removed:
  - `PlayerPreLoginEvent->getPlayer()`
  - `Cancellable->setCancelled()`: this allows implementors of `Cancellable` to implement their own cancellation mechanisms, such as the complex one in `PlayerPreLoginEvent`
- The following API methods have been moved:
  - `Event->isCancelled()` -> `CancellableTrait->isCancelled()`: this was a stub which threw `BadMethodCallException` if the class didn't implement `Cancellable`; now this is simply not available on non-cancellable events
  - `Event->setCancelled()` has been split into `cancel()` and `uncancel()`, and moved to `CancellableTrait`
  - `HandlerList::unregisterAll()` -> `HandlerListManager->unregisterAll()`
  - `HandlerList::getHandlerListFor()` -> `HandlerListManager->getListFor()`
  - `HandlerList::getHandlerLists()` -> `HandlerListManager->getAll()`
- The following API methods have changed behaviour:
  - `PlayerCreationEvent->setPlayerClass()` now verifies that the player class set is instantiable.
- The following classes have been moved:
  - `pocketmine\plugin\RegisteredListener` -> `pocketmine\event\RegisteredListener`

### Inventory
- All crafting and recipe related classes have been moved to the `pocketmine\crafting` namespace.
- The following classes have been added:
  - `CallbackInventoryChangeListener`
  - `CreativeInventory`: contains the creative functionality previously embedded in `pocketmine\item\Item`, see Item changes for details
  - `InventoryChangeListener`: allows listening (but not interfering with) events in an inventory.
  - `PlayerCraftingInventory`: represents the player's own 2x2 crafting grid.
  - `PlayerEnderInventory`: represents the pure storage part of the player's ender inventory, without any block information
  - `TemporaryInventory`: interface which should be implemented by any inventory whose contents should be evacuated when closing.
  - `transaction\CreateItemAction`
  - `transaction\DestroyItemAction`
  - `transaction\TransactionBuilderInventory`: facilitates building `InventoryTransaction`s using standard `Inventory` API methods
- The following classes have been renamed / moved:
  - `ContainerInventory` -> `pocketmine\block\inventory\BlockInventory`
- The following classes have been moved to the `pocketmine\block\inventory` namespace:
  - `AnvilInventory`
  - `ChestInventory`
  - `DoubleChestInventory`
  - `EnchantInventory`
  - `EnderChestInventory`
  - `FurnaceInventory`
- The following classes have been removed:
  - `CustomInventory`
  - `InventoryEventProcessor`
  - `Recipe`
  - `transaction\CreativeInventoryAction`
- The following API methods have been added:
  - `Inventory->addChangeListeners()`
  - `Inventory->getChangeListeners()`
  - `Inventory->removeChangeListeners()`
  - `Inventory->swap()`: swaps the contents of two slots
  - `Inventory->getAddableItemQuantity()`: returns how many items from the given stack can be added to the inventory, used for partial pickups of itemstacks with a full inventory
- The following API methods have been removed:
  - `BaseInventory->getDefaultSize()`
  - `BaseInventory->setSize()`
  - `CraftingGrid->getHolder()`
  - `EnderChestInventory->setHolderPosition()`
  - `Inventory->close()`
  - `Inventory->dropContents()`
  - `Inventory->getName()`
  - `Inventory->getTitle()`
  - `Inventory->onSlotChange()`
  - `Inventory->open()`
  - `Inventory->sendContents()`
  - `Inventory->sendSlot()`
  - `InventoryAction->onExecuteFail()`
  - `InventoryAction->onExecuteSuccess()`
  - `PlayerInventory->sendCreativeContents()`
- The following API methods have signature changes:
  - `BaseInventory->__construct()` no longer accepts a list of items to initialize with.
  - `CraftingGrid->__construct()` no longer accepts a `Player` parameter. 
  - `Inventory->clear()` now returns `void` instead of `bool`.
  - `Inventory->setItem()` now returns `void` instead of `bool`.
  - `InventoryAction->execute()` now returns `void` instead of `bool`.
- `PlayerInventory->setItemInHand()` now sends the update to viewers of the player.
- `CraftingGrid` is now abstract.

### Item
#### General
- A new `VanillaItems` class has been added, which contains static methods for creating any currently-known item type. This should be preferred instead of use of `ItemFactory::get()` where constants were used.
- `StringToItemParser` singleton has been added:
  - This allows mapping any string to any item, irrespective of IDs
  - These mappings are used by `/give` and `/clear`, and are made with custom plugin aliases in mind.
  - Yes, this means you can finally add your own custom aliases to `/give` without ugly hacks!
- `LegacyStringToItemParser` singleton has been added. This supports id:meta parsing in the same way that `ItemFactory::fromString()` used to, but its use is discouraged.
- `ItemFactory` is now a singleton instead of static class, and its remaining methods are no longer static. You can get its instance by `ItemFactory::getInstance()`.
- `Item->count` is no longer public.
- The hierarchy of writable books has been changed: `WritableBook` and `WrittenBook` now extend `WritableBookBase`.
- The following API methods have signature changes:
  - `WritableBookBase->setPages()` now accepts `WritableBookPage[]` instead of `CompoundTag[]`.
  - `ItemFactory->get()` no longer accepts `string` for the `tags` parameter.
- The following methods are now fluent:
  - `WritableBookBase->setPages()`
  - `Item->addEnchantment()`
  - `Item->removeEnchantment()`
  - `Item->removeEnchantments()`
  - `Armor->setCustomColor()`
  - `WrittenBook->setTitle()`
  - `WrittenBook->setAuthor()`
  - `WrittenBook->setGeneration()`
- The following API methods have been removed:
  - `Item->getNamedTagEntry()`
  - `Item->removeNamedTagEntry()`
  - `Item->setDamage()`: "Damage" is now immutable for all items except `Durable` descendents.
  - `Item->setNamedTagEntry()`
  - `Item::get()`: prefer `VanillaItems` or `StringToItemParser` if possible; use `ItemFactory->get()` if you have no other choice
  - `Item::fromString()`: use `StringToItemParser->parse()` or `LegacyStringToItemParser->parse()` instead
  - `ItemFactory::fromString()`
  - `Item->setCompoundTag()`
  - `Item->getCompoundTag()`
  - `Item->hasCompoundTag()`
  - `Potion::getPotionEffectsById()`
  - `ProjectileItem->getProjectileEntityType()`
- The following constants have been removed:
  - `Potion::ALL` - use `PotionType::getAll()` instead
  - `Potion::WATER`
  - `Potion::MUNDANE`
  - `Potion::LONG_MUNDANE`
  - `Potion::THICK`
  - `Potion::AWKWARD`
  - `Potion::NIGHT_VISION`
  - `Potion::LONG_NIGHT_VISION`
  - `Potion::INVISIBILITY`
  - `Potion::LONG_INVISIBILITY`
  - `Potion::LEAPING`
  - `Potion::LONG_LEAPING`
  - `Potion::STRONG_LEAPING`
  - `Potion::FIRE_RESISTANCE`
  - `Potion::LONG_FIRE_RESISTANCE`
  - `Potion::SWIFTNESS`
  - `Potion::LONG_SWIFTNESS`
  - `Potion::STRONG_SWIFTNESS`
  - `Potion::SLOWNESS`
  - `Potion::LONG_SLOWNESS`
  - `Potion::WATER_BREATHING`
  - `Potion::LONG_WATER_BREATHING`
  - `Potion::HEALING`
  - `Potion::STRONG_HEALING`
  - `Potion::HARMING`
  - `Potion::STRONG_HARMING`
  - `Potion::POISON`
  - `Potion::LONG_POISON`
  - `Potion::STRONG_POISON`
  - `Potion::REGENERATION`
  - `Potion::LONG_REGENERATION`
  - `Potion::STRONG_REGENERATION`
  - `Potion::STRENGTH`
  - `Potion::LONG_STRENGTH`
  - `Potion::STRONG_STRENGTH`
  - `Potion::WEAKNESS`
  - `Potion::LONG_WEAKNESS`
  - `Potion::WITHER`
- The following methods have been renamed:
  - `Item->getDamage()` -> `Item->getMeta()`
- The following methods have been moved to `pocketmine\inventory\CreativeInventory`:
  - `Item::addCreativeItem()` -> `CreativeInventory::add()`
  - `Item::clearCreativeItems()` -> `CreativeInventory::clear()`
  - `Item::getCreativeItemIndex()` -> `CreativeInventory::getItemIndex()`
  - `Item::getCreativeItems()` -> `CreativeInventory::getAll()`
  - `Item::initCreativeItems()` -> `CreativeInventory::init()`
  - `Item::isCreativeItem()` -> `CreativeInventory::contains()`
  - `Item::removeCreativeItem()` -> `CreativeInventory::remove()`
- The following classes have been added:
  - `ArmorTypeInfo`
  - `Fertilizer`
  - `LiquidBucket`
  - `MilkBucket`
  - `PotionType`: enum class containing information about vanilla potion types
  - `Releasable`: this interface is implemented by items like bows which have a "release" action
  - `StringToItemParser`: allows converting string IDs into any item, used by `/give` and `/clear`
  - `WritableBookBase`
  - `WritableBookPage`
- The following API methods have been added:
  - `Armor->getArmorSlot()`
  - `Item->canStackWith()`: returns whether the two items could be contained in the same inventory slot, ignoring count and stack size limits
  - `Potion->getType()`: returns a `PotionType` enum object containing information such as the applied effects
  - `ProjectileItem->createEntity()`: returns a new instance of the projectile entity that will be thrown
- The following classes have been removed:
  - `ChainBoots`
  - `ChainChestplate`
  - `ChainHelmet`
  - `ChainLeggings`
  - `DiamondBoots`
  - `DiamondChestplate`
  - `DiamondHelmet`
  - `DiamondLeggings`
  - `GoldBoots`
  - `GoldChestplate`
  - `GoldHelmet`
  - `GoldLeggings`
  - `IronBoots`
  - `IronChesplate`
  - `IronHelmet`
  - `IronLeggings`
  - `LeatherBoots`
  - `LeatherCap`
  - `LeatherPants`
  - `LeatherTunic`

#### NBT handling
- Serialized NBT byte array caches are no longer stored on itemstacks. These caches were a premature optimization used for network layer serialization and as such were dependent on the network NBT format.
- Internal NBT usage has been marginalized. It's no longer necessary to immediately write changes to NBT. The following hooks have been added:
  - `Item->serializeCompoundTag()`
  - `Item->deserializeCompoundTag()`
- It's planned to remove runtime NBT from items completely, but this currently presents unresolved backwards-compatibility problems.

#### Enchantment
- `VanillaEnchantments` class has been added. This exposes all vanilla enchantment types as static methods, replacing the old `Enchantment::get()` nastiness.
  - Example: `Enchantment::get(Enchantment::PROTECTION)` is replaced by `VanillaEnchantments::PROTECTION()`
  - These methods also provide proper type information to static analysers instead of just generic `Enchantment`, making them easier to code with.
- The boundaries between MCPE enchantment IDs and PocketMine-MP internals are now more clear.
  - ID handling is moved to `pocketmine\data\bedrock\EnchantmentIdMap` singleton.
  - All enchantment ID constants have been removed from `Enchantment`. `pocketmine\data\bedrock\EnchantmentIds` if you still need legacy effect IDs for some reason.
- `Enchantment::RARITY_*` constants were moved to `Rarity` class, and the `RARITY_` prefixes removed.
- `Enchantment::SLOT_*` constants were moved to `ItemFlags` class, and the `SLOT_` prefixes removed.
- The following API methods have been moved:
  - `Enchantment::registerEnchantment()` -> `EnchantmentIdMap->register()`
  - `Enchantment::getEnchantment()` -> `EnchantmentIdMap->fromId()`
  - `Enchantment->getId()` -> `EnchantmentIdMap->toId()`
  - `Enchantment::getEnchantmentByName()` -> `StringToEnchantmentParser->parse()`

### Lang
- The following classes have been renamed:
  - `BaseLang` -> `Language`
  - `TranslationContainer` -> `Translatable`
- The following classes have been removed:
  - `TextContainer`
- The following API methods have been added:
  - `Translatable->format()`: allows adding formatting (such as color codes) to a translation
  - `Translatable->prefix()`: allows prefixing formatting
  - `Translatable->postfix()`: allows postfixing formatting
- The following API methods have changed signatures:
  - `Translatable->__construct()` now accepts `array<int|string, Translatable|string>` for parameters, instead of just `list<string>`.
  - `Translatable->getParameter()` now accepts `int|string` for the index instead of just `int`.
  - `Translatable->getParameter()` now returns `Translatable|string` instead of just `string`.
  - `Translatable->getParameters()` now returns `array<int|string, Translatable|string>`.
- `LanguageNotFoundException` has been added. This is thrown when trying to construct a `Language` which doesn't exist in the server files.
- `Translatable` no longer discards keys for translation parameters. Previously, only the insertion order was considered.
- `Translatable` now supports string keys for translation parameters.
- `Translatable` now supports providing other `Translatable`s as translation parameters.
- `Language->translateString()` now supports providing `Translatable`s as translation parameters.
- `Language->translateString()` no longer automatically attempts to translate string parameters. If you want them to be translated, translate them explicitly. This fixes bugs where player chat messages containing translation keys would be unexpectedly translated.
- `Language->translate()` no longer attempts to translate string parameters of `Translatable` (same rationale as previous point).

### Network
- The following fields have been removed:
  - `Network::$BATCH_THRESHOLD`
- The following classes have been added:
  - `NetworkInterfaceStartException`: this may be thrown by `Network->registerInterface()` and `NetworkInterface->start()` to cause a graceful failure without crashing - this should be used when, for example, you are unable to bind a port
- The following classes have been renamed:
  - `SourceInterface` -> `NetworkInterface`
  - `AdvancedSourceInterface` -> `AdvancedNetworkInterface`
- The following classes have been moved:
  - `CompressBatchedTask` -> `mcpe\CompressBatchTask`
  - `level\format\io\ChunkRequestTask` -> `mcpe\ChunkRequestTask`
  - `mcpe\RakLibInterface` -> `mcpe\raklib\RakLibInterface`
- The following classes have been removed:
  - `mcpe\PlayerNetworkSessionAdapter`
- The following methods have been renamed:
  - `UPnP::PortForward()` -> `UPnP::portForward()`
  - `UPnP::RemovePortForward()` -> `UPnP::removePortForward()`
- The following methods have changed signatures:
  - `UPnP::portForward()` now accepts `string $serviceURL, string $internalIP, int $internalPort, int $externalPort`.
  - `UPnP::removePortForward()` now accepts `string $serviceURL, int $externalPort`.
- The following methods have been removed:
  - `NetworkInterface->putPacket()`
  - `NetworkInterface->close()`
  - `NetworkInterface->emergencyShutdown()`
- `NetworkInterface` now represents a more generic interface to be implemented by any network component, as opposed to specifically a player network interface.
- Everything under the `rcon` subnamespace has been removed.
- `upnp\UPnP` has significant changes. It's now a network component instead of a pair of static methods.

### Permission
- The following new permission nodes have been introduced:
  - `pocketmine.group.everyone`: granted to everyone by default
  - `pocketmine.group.operator`: granted to operator players and the console
  These permission nodes can be assigned and overridden by permission attachments just like any other, which means it's now possible to grant **temporary operator** status which goes away when the player disconnects (or the attachment is removed).
- Permissions are now always false if they haven't been set explictly, or granted implicitly by another permission.
- Undefined permissions are now always `false` instead of following the value of `Permission::$DEFAULT_PERMISSION`.
- Permissions internally no longer have default values. Instead, they are now assigned as a child of one of the `pocketmine.group` permissions:
  - `true`: add as child to `pocketmine.group.everyone` with value `true`
  - `false`: do not add to any permission
  - `op`: add as child to `pocketmine.group.operator` with value `true`
  - `notop`: add as child to `pocketmine.group.everyone` with value `true`, and to `pocketmine.group.operator` with value `false`
  However, the `default` key in `plugin.yml` permission definitions continues to be supported.
- Added `PermissibleDelegateTrait` to reduce boilerplate for users of `PermissibleBase`. This trait is used by `ConsoleCommandSender` and `Player`.
- The following API methods have been moved:
  - `Permission::getByName()` -> `PermissionParser::defaultFromString()`
  - `Permission::loadPermissions()` -> `PermissionParser::loadPermissions()`
  - `Permission::loadPermission()` -> `PermissionParser::loadPermission()`
- The following constants have been moved:
  - `Permission::DEFAULT_FALSE` -> `PermissionParser::DEFAULT_FALSE`
  - `Permission::DEFAULT_TRUE` -> `PermissionParser::DEFAULT_TRUE`
  - `Permission::DEFAULT_OP` -> `PermissionParser::DEFAULT_OP`
  - `Permission::DEFAULT_NOT_OP` -> `PermissionParser::DEFAULT_NOT_OP`
- The following API methods have been added:
  - `Permission->addChild()`
  - `Permission->removeChild()`
  - `Permissible->getPermissionRecalculationCallbacks()` - allows reacting to changes of permissions, such as new permissions being granted or denied
  - `Permissible->setBasePermission()` - used for assigning root permissions like `pocketmine.group.operator`; plugins usually shouldn't use this
  - `Permissible->unsetBasePermission()`
  - `PermissionAttachmentInfo->getGroupPermissionInfo()` - returns the `PermissionAttachmentInfo` of the permission that caused the current permission value to be set, or null if the permission is explicit
- The following API methods have been removed:
  - `Permissible->isOp()`: use `Permissible->hasPermission(DefaultPermissions::ROOT_OPERATOR)` instead, **but you really shouldn't directly depend on a player's op status, add your own permissions instead!**
  - `Permissible->setOp()`: use `addAttachment($plugin, DefaultPermissions::ROOT_OPERATOR, true)` instead to add, and `removeAttachment()` to remove it (or addAttachment() with false to explicitly deny it, just like any other permission)
  - `Permission->addParent()`
  - `Permission->getDefault()`
  - `Permission->setDefault()`
  - `PermissionManager->getDefaultPermissions()`
  - `PermissionManager->recalculatePermissionDefaults()`
  - `PermissionManager->subscribeToDefaultPerms()`
  - `PermissionManager->unsubscribeFromDefaultPerms()`
  - `PermissionManager->getDefaultPermSubscriptions()`
  - `PermissionAttachment->getPermissible()`
  - `PermissionAttachmentInfo->getPermissible()`
- The following fields have been removed:
  - `Permission::$DEFAULT_PERMISSION`
- The following API methods have changes:
  - `PermissionParser::defaultFromString()` now throws `InvalidArgumentException` on unknown values.
  - `Permission->__construct()` no longer accepts a `$defaultValue` parameter (see notes above about defaults refactor).you should add your permission as a child of `pocketmine.group.everyone` or `pocketmine.group.operator` instead).
- The following classes have been removed:
  - `ServerOperator`

### Player
- The following classes have been added/moved to the new `pocketmine\player` namespace:
  - `Achievement`
  - `GameMode`
  - `IPlayer`
  - `OfflinePlayer`
  - `PlayerInfo`
  - `Player`
  - `SurvivalBlockBreakHandler`: handles cracking animation, sounds and particles when mining a block in creative
  - `UsedChunkStatus`: enum used internally by the chunk sending system
- The following constants have been removed:
  - `Player::SURVIVAL` - use `GameMode::SURVIVAL()`
  - `Player::CREATIVE` - use `GameMode::CREATIVE()`
  - `Player::ADVENTURE` - use `GameMode::ADVENTURE()`
  - `Player::SPECTATOR` - use `GameMode::SPECTATOR()`
  - `Player::VIEW` - use `GameMode::SPECTATOR()`
- (almost) all packet handlers have been removed from `Player`. They are now encapsulated within the network layer.
- `Player->getSpawn()` no longer returns the world's safe spawn if the player's spawn position isn't set. Returning the safe spawn at the time of call made no sense, because it might not have been safe when actually used. You should pass the result of this function to `World->getSafeSpawn()` to get a safe spawn position instead.
- The following API methods have been added:
  - `Player->attackBlock()`: attack (left click) the target block, e.g. to start destroying it (survival)
  - `Player->attackEntity()`: melee-attack (left click) the target entity (if within range)
  - `Player->breakBlock()`: destroy the target block in the current world (immediately)
  - `Player->consumeHeldItem()`: consume the previously activated item, e.g. eating food
  - `Player->continueBreakBlock()`: punch the target block during destruction in survival, advancing break animation and creating particles
  - `Player->getCurrentWindow()`: returns the inventory window the player is currently viewing, or null if they aren't viewing an inventory
  - `Player->getItemCooldownExpiry()`: returns the tick on which the player's cooldown for a given item expires
  - `Player->getPlayerInfo()`: returns a `PlayerInfo` object containing various metadata about the player
  - `Player->getSaveData()`: returns save data generated on the fly
  - `Player->hasFiniteResources()`
  - `Player->interactBlock()`: interact (right click) the target block in the current world
  - `Player->interactEntity()`: interact (right click) the target entity, e.g. to apply a nametag (not implemented yet)
  - `Player->pickBlock()`: picks (mousewheel click) the target block in the current world
  - `Player->releaseHeldItem()`: release the previously activated item, e.g. shooting a bow
  - `Player->removeCurrentWindow()`: removes the inventory window the player is currently viewing, if any
  - `Player->selectHotbarSlot()`: select the specified hotbar slot
  - `Player->setCurrentWindow()`: sets the inventory the player is currently viewing
  - `Player->stopBreakBlock()`: cease attacking a previously attacked block
  - `Player->toggleFlight()`: tries to start / stop flying (fires events, may be cancelled)
  - `Player->updateNextPosition()`: sets the player's next attempted move location (fires events, may be cancelled)
  - `Player->useHeldItem()`: activate the held item, e.g. throwing a snowball
- The following API methods have been removed:
  - `IPlayer->isBanned()`: this was unreliable because it only checked name bans and didn't account for plugin custom ban systems. Use `Server->getNameBans()->isBanned()` and `Server->getIPBans()->isBanned()` instead.
  - `IPlayer->isOp()`: use `Server` APIs instead
  - `IPlayer->isWhitelisted()`: use `Server->isWhitelisted()` instead
  - `IPlayer->setBanned()`: use `Server` APIs instead
  - `IPlayer->setOp()`: use `Server` APIs instead
  - `IPlayer->setWhitelisted()`: use `Server->setWhitelisted()` instead
  - `Player->addActionBarMessage()`: replaced by `sendActionBarMessage()`
  - `Player->addSubTitle()`: replaced by `sendSubTitle()`
  - `Player->addTitle()`: replaced by `sendTitle()`
  - `Player->addWindow()`: use `Player->setCurrentWindow()` instead
  - `Player->dataPacket()`: replaced by `NetworkSession->sendDataPacket()`
  - `Player->getAddress()`: replaced by `NetworkSession->getIp()`
  - `Player->getPing()`: moved to `NetworkSession`
  - `Player->getPort()`: moved to `NetworkSession`
  - `Player->getWindow()`: use `Player->getCurrentWindow()` instead
  - `Player->getWindowId()`
  - `Player->removeAllWindows()`
  - `Player->removeWindow()`: use `Player->removeCurrentWindow()` instead
  - `Player->sendDataPacket()`: replaced by `NetworkSession->sendDataPacket()`
  - `Player->setCraftingGrid()`: crafting tables now work the same way as other containers; use `Player->setCurrentWindow()`
  - `Player->updateNextPosition()`: use `Player->handleMovement()` instead
  - `Player->updatePing()`: moved to `NetworkSession`

### Plugin
- API version checks are now more strict. It is no longer legal to declare multiple minimum versions on the same major version. Doing so will now cause the plugin to fail to load with the message `Multiple minimum API versions found for some major versions`.
- `plugin.yml` YAML commands loading is now internalized inside `PluginBase`.
- `PluginManager->registerEvent()` now has a simpler signature: `registerEvent(string $event, \Closure $handler, int $priority, Plugin $plugin, bool $handleCancelled = false)`. The provided closure must accept the specified event class as its only parameter. See [Event API changes](#event) for more details.
- The following classes have been removed:
  - `PluginLogger`
- The following constants have been removed:
  - `PluginLoadOrder::STARTUP` - use `PluginEnableOrder::STARTUP()` 
  - `PluginLoadOrder::POSTWORLD` - use `PluginEnableOrder::POSTWORLD()`
- The following interface requirements have been removed:
  - `Plugin->onEnable()`: this is now internalized inside `PluginBase`
  - `Plugin->onDisable()`: same as above
  - `Plugin->onLoad()`: same as above
  - `Plugin->getServer()` is no longer required to be implemented. It's implemented in `PluginBase` for convenience.
  - `Plugin->isDisabled()` was removed (use `Plugin->isEnabled()` instead).
  - `Plugin` no longer extends `CommandExecutor`. This means that `Plugin` implementations don't need to implement `onCommand()` anymore.
- The following hook methods have changed visibility:
  - `PluginBase->onEnable()` changed from `public` to `protected`
  - `PluginBase->onDisable()` changed from `public` to `protected`
  - `PluginBase->onLoad()` changed from `public` to `protected`
- The following hook methods have been renamed:
  - `Plugin->setEnabled()` -> `Plugin->onEnableStateChange()`. This change was made to force plugin developers misusing this hook to stop, and to give it a name that better describes what it does.
- The following API methods have been removed:
  - `PluginManager->addPermission()`: use `PermissionManager` instead
  - `PluginManager->callEvent()`: use `Event->call()` instead
  - `PluginManager->getDefaultPermSubscriptions()`: use `PermissionManager` instead
  - `PluginManager->getDefaultPermissions()`: use `PermissionManager` instead
  - `PluginManager->getPermission()`: use `PermissionManager` instead
  - `PluginManager->getPermissionSubscriptions()`: use `PermissionManager` instead
  - `PluginManager->getPermissions()`: use `PermissionManager` instead
  - `PluginManager->loadPlugin()`: use `PluginManager->loadPlugins()` instead
  - `PluginManager->recalculatePermissionDefaults()`: use `PermissionManager` instead
  - `PluginManager->removePermission()`: use `PermissionManager` instead
  - `PluginManager->subscribeToDefaultPerms()`: use `PermissionManager` instead
  - `PluginManager->subscribeToPermission()`: use `PermissionManager` instead
  - `PluginManager->unsubscribeFromDefaultPerms()`: use `PermissionManager` instead
  - `PluginManager->unsubscribeFromPermission()`: use `PermissionManager` instead
- The following API methods have changed behaviour:
  - `PluginManager->loadPlugins()` now accepts paths to files as well as directories, in which case it will load only the plugin found in the target file.
- It is no longer permitted to throw exceptions from `PluginBase->onEnable()` or `PluginBase->onLoad()`. Doing so will now cause the server to crash.

### Promise
A very basic in-house implementation of Promises has been added. This is currently used for handling world generation requests.

- `PromiseResolver` is created by the creator of the task. The task should call `PromiseResolver->resolve()` when the result is ready.
- `Promise` can be obtained by using `PromiseResolver->getPromise()` and should be returned to API consumers.

Please note that this was not written with plugins in mind and its API may change in a future version.

### Scheduler
#### Thread-local storage for AsyncTasks
- TLS has been completely rewritten in this release to be self contained, more robust and easier to use.
- Now behaves more like simple properties. `storeLocal()` writes, `fetchLocal()` reads.
- Self-contained and doesn't depend on the async pool to clean up after it.
- Values are automatically removed from storage when the `AsyncTask` is garbage-collected, just like a regular property.
- Supports storing multiple values, differentiated by string names.
- `fetchLocal()` can now be used multiple times. It no longer deletes the stored value.
- The following classes have been removed:
  - `FileWriteTask`
- The following methods have been removed:
  - `AsyncTask->peekLocal()`: use `fetchLocal()` instead
- The following methods have signature changes:
  - `AsyncTask->storeLocal()` now has the signature `storeLocal(string $key, mixed $complexData) : void`
  - `AsyncTask->fetchLocal()` now has the signature `fetchLocal(string $key) : mixed`

#### Other AsyncTask changes
- `AsyncPool` uses a new, significantly more performant algorithm for task collection.
- `BulkCurlTask` has had the `$complexData` constructor parameter removed.
- `BulkCurlTask->__construct()` now accepts `BulkCurlTaskOperation[]` instead of `mixed[]`.
- `pocketmine\Collectable` has been removed, and is no longer extended by `AsyncTask`.
- The following hooks have been added:
  - `AsyncTask->onError()`: called on the main thread when an uncontrolled error was detected in the async task, such as a memory failure
- The following hooks have signature changes:
  - `AsyncTask->onCompletion()` no longer accepts a `Server` parameter, and has a `void` return type.
  - `AsyncTask->onProgressUpdate()` no longer accepts a `Server` parameter, and has a `void` return type.
- The following API methods have been removed:
  - `AsyncTask->getFromThreadStore()`: use `AsyncTask->worker->getFromThreadStore()`
  - `AsyncTask->removeFromThreadStore()`: use `AsyncTask->worker->removeFromThreadStore()`
  - `AsyncTask->saveToThreadStore()`: use `AsyncTask->worker->saveToThreadStore()`

#### Non-AsyncTask changes
- Added `CancelTaskException`, which can be thrown from `Task->onRun()` to cancel a task (especially useful for `ClosureTask`).
- The `$currentTick` parameter of `Task->onRun()` has been removed (use `Server->getTick()` instead if needed).
- Callables given to `ClosureTask` are no longer required to declare a `void` typehint (useful for arrow functions).

### Server
- New chat broadcasting APIs have been implemented, which don't depend on the permission system.
  - The following API methods have been added:
    - `Server->subscribeToBroadcastChannel()` - allows subscribing a `CommandSender` to receive chat messages (and other message types) on any channel
    - `Server->unsubscribeFromBroadcastChannel()`
    - `Server->unsubscribeFromAllBroadcastChannels()`
    - `Server->getBroadcastChannelSubscribers()`
  - Giving `Player` any `pocketmine.broadcast.*` permissions will cause them to automatically subscribe to the corresponding broadcast channel (and removing them will unsubscribe it).
  - It's now possible to create and subscribe to custom broadcast channels without using permissions.
  - However, `Player`s may automatically unsubscribe themselves from the builtin broadcast channels if they don't have the proper permissions.
  - Automatic subscribe/unsubscribe from custom broadcast channels can be implemented using the new `Permissible` permission recalculation callbacks API.
- The following API methods have been added:
  - `Server->getIpV6()`
  - `Server->getPortV6()`
- The following API methods have been removed:
  - `Server->reloadWhitelist()`
  - `Server->getLevelMetadata()`
  - `Server->getPlayerMetadata()`
  - `Server->getEntityMetadata()`
  - `Server->getDefaultGamemode()`
  - `Server->getLoggedInPlayers()`
  - `Server->onPlayerLogout()`
  - `Server->addPlayer()`
  - `Server->removePlayer()`
  - `Server->reload()`
  - `Server->getSpawnRadius()`
  - `Server->enablePlugin()`
  - `Server->disablePlugin()`
  - `Server->getGamemodeString()` - replaced by `pocketmine\player\GameMode->getTranslationKey()`
  - `Server->getGamemodeName()` - replaced by `pocketmine\player\GameMode->name()`
  - `Server->getGamemodeFromString()` - replaced by `GameMode::fromString()`
  - `Server->broadcast()` - use `Server->broadcastMessage()` instead
- The following API methods have changed:
  - `Server->getOfflinePlayerData()` no longer creates data when it doesn't exist.
- The following API methods have been renamed:
  - `Server->getPlayer()` -> `Server->getPlayerByPrefix()` (consider using `Server->getPlayerExact()` instead where possible)

### Level / World
#### General
- All references to `Level` in the context of "world" have been changed to `World`.
  - The `pocketmine\level` namespace has been renamed to `pocketmine\world`
  - All classes containing the world `Level` in the name in the "world" context have been changed to `World`.
  - `Position->getLevel()` has been renamed to `Position->getWorld()`, and `Position->level` has been renamed to `Position->world`.
- Extracted a `WorldManager` unit from `Server`
  - `Server->findEntity()` -> `WorldManager->findEntity()`
  - `Server->generateLevel()` -> `WorldManager->generateWorld()`
  - `Server->getAutoSave()` -> `WorldManager->getAutoSave()`
  - `Server->getDefaultLevel()` -> `WorldManager->getDefaultWorld()`
  - `Server->getLevel()` -> `WorldManager->getWorld()`
  - `Server->getLevelByName()` -> `WorldManager->getWorldByName()`
  - `Server->getLevels()` -> `WorldManager->getWorlds()`
  - `Server->isLevelGenerated()` -> `WorldManager->isWorldGenerated()`
  - `Server->isLevelLoaded()` -> `WorldManager->isWorldLoaded()`
  - `Server->loadLevel()` -> `WorldManager->loadWorld()`
    - `WorldManager->loadWorld()` may convert worlds if requested (the `$autoUpgrade` parameter must be provided).
  - `Server->setAutoSave()` -> `WorldManager->setAutoSave()`
  - `Server->setDefaultLevel()` -> `WorldManager->setDefaultWorld()`
  - `Server->unloadLevel()` -> `WorldManager->unloadWorld()`
- The following static classes have been un-static-ified and converted to singletons (use `Whatever::getInstance()->method()` instead of `Whatever::method()`):
  - `GeneratorManager`
  - `WorldProviderManager`
- The following classes have been added:
  - `BlockTransaction`: allows creating batch commits of block changes with validation conditions - if any block can't be applied, the whole transaction fails to apply.
  - `ChunkListenerNoOpTrait`: contains default no-op stubs for chunk listener implementations
  - `ChunkListener`: interface allowing subscribing to events happening on a given chunk
  - `ChunkLockId`: used by `World->lockChunk()` and `World->unlockChunk()`
  - `TickingChunkLoader`: a `ChunkLoader` specialization that allows ticking chunks
  - `format\io\FastChunkSerializer`: provides methods to encode a chunk for transmitting to another thread
  - `WorldCreationOptions`: used for passing world generator options to `WorldManager->generateWorld()`
- `ChunkLoader` no longer requires implementing `getX()` and `getZ()`.
- `ChunkLoader` no longer causes chunks to get random updates. If this behaviour is needed, implement `TickingChunkLoader`.
- The following classes have been renamed:
  - `pocketmine\world\utils\SubChunkIteratorManager` -> `pocketmine\world\utils\SubChunkExplorer`
- The following class constants have been added:
  - `Chunk::COORD_BIT_SIZE`
  - `Chunk::COORD_MASK`
  - `Chunk::DIRTY_FLAG_BLOCKS`
  - `Chunk::DIRTY_FLAG_TERRAIN`
  - `Chunk::EDGE_LENGTH`
  - `SubChunk::COORD_BIT_SIZE`
  - `SubChunk::COORD_MASK`
  - `SubChunk::EDGE_LENGTH`
  - `World::Y_MIN`
- The following API methods have been added:
  - `WorldManager->getAutoSaveTicks()`
  - `WorldManager->setAutoSaveTicks()`
  - `World->notifyNeighbourBlockUpdate()`
  - `World->registerChunkListener()`
  - `World->unregisterChunkListener()`
  - `World->getBlockAt()` (accepts int x/y/z instead of Vector3, faster for some use cases)
  - `World->setBlockAt()` (accepts int x/y/z instead of Vector3, faster for some use cases)
  - `Chunk->isDirty()` (replacement for `Chunk->hasChanged()`)
  - `Chunk->getDirtyFlag()` (more granular component-based chunk dirty-flagging, used to avoid saving unmodified parts of the chunk)
  - `Chunk->setDirty()`
  - `Chunk->setDirtyFlag()`
- The following API methods have been removed from the public API:
  - `Chunk->addEntity()`
  - `Chunk->fastSerialize()` (use `FastChunkSerializer::serializeTerrain()` instead)
  - `Chunk->getBlockData()`
  - `Chunk->getBlockDataColumn()`
  - `Chunk->getBlockId()`
  - `Chunk->getBlockIdColumn()`
  - `Chunk->getBlockLight()`
  - `Chunk->getBlockLightColumn()`
  - `Chunk->getBlockSkyLight()`
  - `Chunk->getBlockSkyLightColumn()`
  - `Chunk->getEntities()`
  - `Chunk->getMaxY()`
  - `Chunk->getSavableEntities()`
  - `Chunk->getSubChunkSendCount()` (this was specialized for protocol usage)
  - `Chunk->getX()`
  - `Chunk->getZ()`
  - `Chunk->hasChanged()` (use `Chunk->isDirty()` or `Chunk->getDirtyFlag()` instead)
  - `Chunk->isGenerated()`
  - `Chunk->networkSerialize()` (see `ChunkSerializer` in the `network\mcpe\serializer` package)
  - `Chunk->populateSkyLight()` (use `SkyLightUpdate->recalculateChunk()` instead)
  - `Chunk->recalculateHeightMap()` (moved to `SkyLightUpdate`)
  - `Chunk->recalculateHeightMapColumn()` (moved to `SkyLightUpdate`)
  - `Chunk->removeEntity()`
  - `Chunk->setAllBlockLight()`
  - `Chunk->setAllBlockSkyLight()`
  - `Chunk->setBlock()`
  - `Chunk->setBlockData()`
  - `Chunk->setBlockId()`
  - `Chunk->setBlockLight()`
  - `Chunk->setBlockSkyLight()`
  - `Chunk->setChanged()` (use `Chunk->setDirty()` or `Chunk->setDirtyFlag()` instead)
  - `Chunk->setGenerated()`
  - `Chunk->setX()`
  - `Chunk->setZ()`
  - `Chunk::fastDeserialize()` (use `FastChunkSerializer::deserializeTerrain()` instead)
  - `ChunkLoader->getLevel()`
  - `ChunkLoader->getLoaderId()` (now object ID is used)
  - `ChunkLoader->getPosition()`
  - `ChunkLoader->isLoaderActive()`
  - `World->addChunkPacket()`
  - `World->addGlobalPacket()`
  - `World->broadcastGlobalPacket()`
  - `World->broadcastLevelEvent()`
  - `World->broadcastLevelSoundEvent()`
  - `World->checkSpawnProtection()`
  - `World->generateChunkCallback()`
  - `World->getBlockDataAt()`
  - `World->getBlockIdAt()`
  - `World->getBlockSkyLightAt()` (use `World->getRealBlockSkyLightAt()` or `World->getPotentialBlockSkyLightAt()`, depending on use-case)
  - `World->getChunkTiles()`
  - `World->getFullBlock()`
  - `World->getHeightMap()` (misleading name, only actually useful for sky light calculation - you probably want `getHighestBlockAt()` instead)
  - `World->getTickRate()`
  - `World->getTileById()`
  - `World->isFullBlock()`
  - `World->isFullBlock()` (use `Block->isFullCube()` instead)
  - `World->sendBlocks()`
  - `World->sendTime()`
  - `World->setBlockDataAt()`
  - `World->setBlockIdAt()`
  - `World->setBlockLightAt()`
  - `World->setBlockSkyLightAt()`
  - `World->setHeightMap()` (misleading name, only actually useful for sky light calculation)
  - `World->setTickRate()`
  - `World->updateBlockLight()`
  - `World->updateSkyLight()`
  - `World::generateChunkLoaderId()`
- The following API methods have changed signatures:
  - `Chunk->__construct()` now has the signature `array<int, SubChunk> $subChunks, BiomeArray $biomeIds, bool $terrainPopulated`.
  - `Chunk->getSubChunk()` now returns `SubChunk` instead of `SubChunkInterface|null` (and throws `InvalidArgumentException` on out-of-bounds coordinates).
  - `Chunk->getSubChunks()` now returns `array<int, SubChunk>` instead of `SplFixedArray<SubChunk>`.
  - `Chunk->setSubChunk()` no longer accepts `SubChunkInterface`, and the `$allowEmpty` parameter has been removed.
  - `ChunkManager->setChunk()` (and its notable implementations in `World` and `SimpleChunkManager`) no longer accepts NULL for the `$chunk` parameter.
  - `GeneratorManager->registerGenerator()` now requires a `\Closure $presetValidator` parameter. This is used to check generator options of worlds and configs before attempting to use them.
  - `Position->__construct()` now requires the `$world` parameter (it's no longer optional). 
  - `World->addParticle()` now has the signature `addParticle(Vector3 $pos, Particle $particle, ?Player[] $players = null) : void`
  - `World->addRandomTickedBlock()` now accepts `Block` instead of `int, int`.
  - `World->addSound()` now has the signature `addSound(?Vector3 $pos, Sound $sound, ?Player[] $players = null) : void`
  - `World->getChunk()` no longer accepts a `$create` parameter.
  - `World->getRandomTickedBlocks()` now returns `bool[]` instead of `SplFixedArray`.
  - `World->loadChunk()` now returns `?Chunk`, and the `$create` parameter has been removed.
  - `World->removeRandomTickedBlock()` now accepts `Block` instead of `int, int`.
  - `World->setBlock()` has had the `$direct` parameter removed.
  - `World->setChunks()` no longer accepts a `$deleteEntitiesAndTiles` parameter.
  - `World->updateAllLight()` now accepts `int, int, int` instead of `Vector3`.
  - `WorldManager->generateWorld()` (previously `Server->generateWorld()`) now accepts `WorldCreationOptions` instead of `int $seed, class-string<Generator> $generator, mixed[] $options`
  - `World->lockChunk()` now requires `ChunkLockId $lockId` parameter.
  - `World->unlockChunk()` now requires a `?ChunkLockId $lockId` parameter. If a non-null lockID is given, the lock on the chunk will only be removed if it matches the given lockID.
  - `World->unlockChunk()` now returns `bool` instead of `void` (to signal whether unlocking succeded or not).
- The following API methods have been renamed / moved:
  - `World->getChunks()` -> `World->getLoadedChunks()`
  - `World->getCollisionCubes()` -> `World->getCollisionBoxes()`
  - `World->getName()` -> `World->getDisplayName()`
  - `World->populateChunk()` has been split into `World->requestChunkPopulation()` and `World->orderChunkPopulation()`.
- The following API methods have changed behaviour:
  - `World->getAdjacentChunks()` now returns an array indexed using `World::chunkHash()`, where the `x` and `z` components are the relative offsets from the target chunk (range -1 to +1).
  - `World->getChunk()` no longer tries to load chunks from disk. If the chunk is not already in memory, null is returned. (This behaviour now properly matches other `ChunkManager` implementations.)
  - `World->getHighestBlockAt()` now returns `null` instead of `-1` if the target X/Z column contains no blocks.
  - The following methods now throw `WorldException` when targeting ungenerated terrain:
    - `World->getSafeSpawn()` (previously it just silently returned the input position)
    - `World->getHighestBlockAt()` (previously it returned -1)
  - `World->loadChunk()` no longer creates an empty chunk when the target chunk doesn't exist on disk.
  - `World->setChunk()` has the following behavioural changes:
    - Now fires `ChunkLoadEvent` and `ChunkListener->onChunkLoaded()` when replacing a chunk that didn't previously exist.
    - Now updates entities in the replaced chunk and its neighbours. This fixes bugs such as paintings not dropping and dropped items floating in midair if the ground was lower than before.
    - Entities are no longer deleted on chunk replacement.
    - Tiles are no longer deleted on chunk replacement, unless one of the following conditions is met:
      - the target block in the new chunk doesn't expect a tile
      - the target block in the new chunk expects a different type of tile (responsibility of the plugin developer to create the new tile)
      - there's already a tile in the target chunk which conflicts with the old one
  - `World->useBreakOn()` now returns `false` when the target position is in an ungenerated or unloaded chunk (or chunk locked for generation).
  - `World->useItemOn()` now returns `false` when the target position is in an ungenerated or unloaded chunk (or chunk locked for generation).
- A `ChunkListener` interface has been extracted from `ChunkLoader`. The following methods have been moved:
  - `ChunkLoader->onBlockChanged()` -> `ChunkListener->onBlockChanged()`
  - `ChunkLoader->onChunkChanged()` -> `ChunkListener->onChunkChanged()`
  - `ChunkLoader->onChunkLoaded()` -> `ChunkListener->onChunkLoaded()`
  - `ChunkLoader->onChunkPopulated()` -> `ChunkListener->onChunkPopulated()`
  - `ChunkLoader->onChunkUnloaded()` -> `ChunkListener->onChunkUnloaded()`
- `Location` has been moved to `pocketmine\entity\Location`.

#### Particles
- `DestroyBlockParticle` has been renamed to `BlockBreakParticle` for consistency.
- `DustParticle->__construct()` now accepts a `pocketmine\color\Color` object instead of `r, g, b, a`.
- `pocketmine\world\particle\Particle` no longer extends `pocketmine\math\Vector3`, and has been converted to an interface.
- Added the following `Particle` classes:
  - `DragonEggTeleportParticle`
  - `PunchBlockParticle`

#### Sounds
- `pocketmine\world\sound\Sound` no longer extends `pocketmine\math\Vector3`, and has been converted to an interface.
- `Sound->encode()` now accepts `pocketmine\math\Vector3`.
- Added the following classes:
  - `ArrowHitSound`
  - `BlockBreakSound`
  - `BlockPlaceSound`
  - `BowShootSound`
  - `BucketEmptyLavaSound`
  - `BucketEmptyWaterSound`
  - `BucketFillLavaSound`
  - `BucketFillWaterSound`
  - `ChestCloseSound`
  - `ChestOpenSound`
  - `EnderChestCloseSound`
  - `EnderChestOpenSound`
  - `ExplodeSound`
  - `FlintSteelSound`
  - `ItemBreakSound`
  - `NoteInstrument`
  - `NoteSound`
  - `PaintingPlaceSound`
  - `PotionSplashSound`
  - `RedstonePowerOffSound`
  - `RedstonePowerOnSound`
  - `ThrowSound`
  - `XpCollectSound`
  - `XpLevelUpSound`

### Utils
- The `Color` class was removed. It's now found as `pocketmine\color\Color` in the [`pocketmine/color`](https://github.com/pmmp/Color) package.
- The `UUID` class was removed. [`ramsey/uuid`](https://github.com/ramsey/uuid) version 4.1 is now used instead.
  - `UUID::fromData()` can be replaced by `Ramsey\Uuid\Uuid::uuid3()`
  - `UUID::fromRandom()` can be replaced by `Ramsey\Uuid\Uuid::uuid4()`
  - `UUID::fromBinary()` can be replaced by `Ramsey\Uuid\Uuid::fromBytes()` (use `Ramsey\Uuid\Uuid::isValid()` to check validity)
  - `UUID::toBinary()` is replaced by `Ramsey\Uuid\UuidInterface::getBytes()`
  - See the [documentation for `ramsey/uuid`](https://uuid.ramsey.dev/en/latest/introduction.html) for more information.
- `Terminal::hasFormattingCodes()` no longer auto-detects the availability of formatting codes. Instead it's necessary to use `Terminal::init()` with no parameters to initialize, or `true` or `false` to override.
- `Config->save()` no longer catches exceptions thrown during emitting to disk.
- The following new classes have been added:
  - `InternetException`
  - `Internet`
  - `Process`
- The following API methods have been added:
  - `Config->getPath()`: returns the path to the config on disk
  - `Config::parseList()`: parses a list of entries like `ops.txt` into an array
  - `Config::parseProperties()`: parses a properties file like `server.properties` into an array
  - `Config::writeList()`
  - `Config::writeProperties()`
  - `Terminal::write()`: emits a Minecraft-formatted text line without newline
  - `Terminal::writeLine()`: emits a Minecraft-formatted text line with newline
  - `Utils::recursiveUnlink()`: recursively deletes a directory and its contents
- The following API class constants have been added:
  - `TextFormat::COLORS`: lists all known color codes
  - `TextFormat::FORMATS`: lists all known formatting codes (e.g. italic, bold). (`RESET` is not included because it _removes_ formats, rather than adding them.)
- The following deprecated API redirects have been removed:
  - `Utils::execute()`: moved to `Process`
  - `Utils::getIP()`: moved to `Internet`
  - `Utils::getMemoryUsage()`: moved to `Process`
  - `Utils::getRealMemoryUsage()`: moved to `Process`
  - `Utils::getThreadCount()`: moved to `Process`
  - `Utils::getURL()`: moved to `Internet`
  - `Utils::kill()`: moved to `Process`
  - `Utils::postURL()`: moved to `Internet`
  - `Utils::simpleCurl()`: moved to `Internet`
- The following API fields have been removed / hidden:
  - `Utils::$ip`
  - `Utils::$online`
  - `Utils::$os`
- The following API methods have signature changes:
  - `Internet::simpleCurl()` now requires a `Closure` for its `onSuccess` parameter instead of `callable`.
  - `Process::kill()` now requires an additional `bool $subprocesses` parameter.
- The following API methods have behavioural changes:
  - `Utils::parseDocComment()` now allows `-` in tag names.
- The following API methods have been removed:
  - `TextFormat::toJSON()`
  - `Utils::getCallableIdentifier()`
- `MainLogger` now pushes log messages to `server.log` before calling logger attachments. This fixes messages not being written to disk when an uncaught error is thrown from a logger attachment.

## Gameplay
### World loading
- Chunks are now sent in proper circles. This improves the experience when flying quickly parallel to X or Z axis, since now more chunks in front of the player will load sooner.
- Many bugs in player respawning have been fixed, including:
  - Spawning underneath bedrock when spawn position referred to ungenerated terrain
  - Spawning underneath bedrock on first server join on very slow machines (or when the machine was under very high load)
  - Spawning inside blocks (or above the ground) when respawning with a custom spawn position set
  - Player spawn positions sticking to the old location when world spawn position changed - this was because the world spawn at time of player creation was used as the player's custom spawn, so the bug will persist for older player data, but will work as expected for new players.

### Blocks
- Implemented the following blocks:
  - bamboo
  - bamboo sapling
  - barrel
  - barrier
  - blast furnace
  - blue ice
  - carved pumpkin
  - coral block
  - daylight sensor
  - dried kelp
  - elements (from Minecraft: Education Edition)
  - hard (stained and unstained) glass (from Minecraft: Education Edition)
  - hard (stained and unstained) glass pane (from Minecraft: Education Edition)
  - jukebox
  - note block
  - red, green, blue and purple torches (from Minecraft: Education Edition)
  - sea pickle
  - slime
  - smoker
  - underwater torches (from Minecraft: Education Edition)
  - additional wood variants of the following:
    - buttons
    - pressure plates
    - signs
    - trapdoors
  - stairs of the following materials:
    - andesite (smooth and natural)
    - diorite (smooth and natural)
    - end stone
    - end stone brick
    - granite (smooth and natural)
    - mossy cobblestone
    - prismarine (natural, dark and bricks)
    - red nether brick
    - red sandstone (and variants)
  - stone-like slabs of many variants
- Non-player entities now bounce when falling on beds.
- Players and mobs now receive reduced fall damage when falling on beds.
- Fixed cake block desync when attempting to eat in creative (eating in creative is not yet supported, but the block rollback was missing).
- Fixed the bounding box of skulls when mounted on a wall.
- Fixed podzol dropping itself when mined (instead of dirt).

### Items
- Implemented the following items:
  - records
  - compounds (from Minecraft: Education Edition)
  - black, brown, blue and white dyes
- Compasses now point to the correct (current) world's spawn point after teleporting players to a different world. Previously, they would continue to point to the spawn of the world that the player initially spawned in.

### Inventory
- Implemented offhand inventory.
- Block-picking is now supported in survival mode.
- Block picking behaviour now matches vanilla (no longer overwrites held item, jumps to existing item where possible).
- Armor can now be equipped by right-clicking while holding it.
- Picking up some items from a dropped stack of items is now supported. This fixes various bugs with being unable to pick up items with an almost-full inventory.
- Fixed arrows getting added to creative players' inventories when picked up.

### Misc
- Added support for emotes.