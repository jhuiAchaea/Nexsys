# Nexsys
Curing System for Achaea Nexus Client

This is an initial release just to get some Nexus code out there.  It's far from its final state, but hopefully people will use it.

Nexus Client Recommendations:
- use small scrollback buffer size (200 or so).  System shouldn't bog browser down, but a large buffer will.
- use logger when you want to be able to see stuff in your buffer
- right click the Nexus GUI and hide most of the stuff that you don't care for (it'll save the settings for next time you login)
- I hide the battlerage buttons and health gauges so there's more room for the game console
- move the tabs around until you like a style.  I use chat on the full left side, map on bottom right, and character tab in top right.


## Install

Download nxs file from: 

Open Nexus Client Settings and Import file.

Type 'load' to load the system.  It's just an alias for run_function("onLoad", {}, "ALL"), so won't have to type load every time.

Set your command separator that you want to use in game (not what you want to use in Nexus)

It should spam some commands on startup every time to grab all serverside settings so the system will know what it needs to change. (Just think of it as resyncing).

System should automatically unpause after startup is complete.

## How to use the system

The goal of the system is to only have to modify tables or specific logic sections if you need to make changes, so don't worry about understanding it completely.

Other Packages will be released that use this system so also look at those to understand how to use it.

### System Settings

Look under system->system and modify the settings there

### Aff Settings

Look under affs->affTable and modify serverside curing prios there

### Balance Settings

If you need to add your own balance, add to balanceTable and follow the pattern for other balances

### Def Settings

Look under defs->defTable and add defs if you need.  Modify the keepup and static prio objects at the bottom to change your prios.  Use defService to create aliases to change your prios on the fly in game.

### Cure Settings

Look under cures->cureTable to see how cures are handled.  Most are set to prio 0 because they are handled by serverside curing.  Class cures like fitness, dragonheal, etc, start at prio 0, but are moved to a different prio when they are needed (see serverside->PrioritySwapping).

### Cache Settings

Look under cache->cacheTable and modify the amount you want to precache for each herb/mineral there.

### Lust

Look under lust->lust and set your whitelist there.  Otherwise it should reject everyone else.

### GMCP

This system tries to mimic the Mudlet GMCP object.  'display GMCP' should print GMCP data in the console.

### Echos

Echo is a wrapper that uses display_notice, but feels easier to remember and can use it for more functionality.

4 base classes exist, Echo, EchoWithPrefix, EchoLine, and EchoLineWithPrefix.

Use client.echo('your echo here') to just display a generic echo with no line breaks.

client.echoLine will do the same but add a newline at the end.

Rather than echoing stuff all over the place, I tried to centralize the echo'ing in echoEvents.  My goal by making it event based was to make it easier to completely reskin your echoing without having to track down every location an echo was used.

You can still echo your own stuff in place, but you might get an idea how you want to do it by checking echoClasses, echoEvents, and echoService out.

### Client Overrides

client.display_notice is overriden using Tael's modified display_notice.

client.generate_text_block is overriden to replace the nexus prompt with a custom prompt if custom prompt is enabled.

It also modifies two things with the default logger: 1) it does not log gagged lines (ew spam) 2) it logs the prompt (nexus logger decided not to log prompts).

#### promptOverride

promptOverride holds all things related to custom prompt.  It essentially uses display_notice to build up a prompt.  This prompt is built off of the client.prompt.vars object.  These variables get set by listening to events and changing them there.  This is safer than being reliant on a function to exist to return us prompt information.  Take a look, it should be relatively easy to add your own stuff.

### commonTables

has a few useful tables/maps you might use in your own code

### serverside

#### serversideEvents

Responsible for capturing serverside settings.  It is also responsible for listening to system changes such as defence/affliction priority changes and determining whether they need to be input to Achaea to change a serverside setting.

#### ServersidePrioritySwapping

Does the logic to perform priority swapping.  Check it out.

#### PrioSwapTrackables

This will include stuff we need to track to help us determine priority swaps.  Like being snapped/vivi modes/heartseed modes/etc.  It's very incomplete.

### utility

This contains the functions we'll use in a lot of places.

client.sendCmd (same as send_command, but we raise an event with it, so we could add listeners elsewhere to track if we sent something or not, or could deny input on aeon and such)

client.psend (send if not paused)

client.rsend (send if not paused and not in slow mode)

client.sendInline (pass array of commands in and this function will separate them with your in-game command separator)

client.timeDiffNow(prev) returns time in milliseconds (pretty much everything that is tracked has a got/lost time in the system, so this function can be used to get the difference between then and now)

aliases to help with system:

load - runs the onLoad function

display - will show you an object in console window..haven't made it be able to show nested objects yet, but that should be easy enough (ex: 'display GMCP' 'display client' 'display Affs'

event - can raise an event that doesn't need data to test stuff (ex: 'event asthmaGotAffEvent')

loggingon/off - will display events/event data to console log.  It's spammy but can help debug.

### queueing

This contains queue classes that help us interact with the in game queueing system more effectively.

client.EqBalQueue is the only one currently.  It adds to your EqBal queue in game, but more importantly it auto-cancels your queue after a configurable time (set to .5 sec atm), so you can safely spam and not worry about having to cancel it if you decide you don't want to do it.  You just need to take your finger off the enter button for .5 sec.

You can chain commands or add them one by one.  .send() clears it, so the next .add starts a fresh set of commands.

example usages:
```
client.EqBalQueue.add('stand').add('dsl ' +client.sys.target+ ' curare prefarar').add('engage ' +client.sys.target).send();
```
or
```
client.EqBalQueue.add('stand');
client.EqBalQueue.add('dsl ' +client.sys.target+ ' curare prefarar');
client.EqBalQueue.add('engage ' +client.sys.target);
client.EqBalQueue.send();
```

#### dor

dor command useful for repeating actions every 1 second.  Also queues it up to EqBal queue.

dora command is 'dor' but it will repeat an alias every 1 second.

#### doCommandWhenCan

this will be where we want to add functionality like 'dv' (diag when we can).  You can implement your own stuff here and perform your own logic to determine if you can do it or not.  The system will run getCommandsToDo in its normal output.


### gags

Common gags are in here.  Nexus does not show prompt if all lines in a block are gagged, so no need for a deleteLine + prompt function.

## TODO List

- retard detection
- block commands on aeon
- defences/cures/balances for all classes
- more user friendly ways to interact with systm out of the box
- more custom prompt options/easier way to set custom prompt than in code
- way to map setting configurations to nexus variables so can store/load from server so when people upgrade, they won't lose previously configured settings





