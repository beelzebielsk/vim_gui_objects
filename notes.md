- Don't make a bunch of specialized widgets. Not necessary. Make a
  widget wrapper with a keypress function. When in normal mode, it'll
  handle navigation. When in 'insert' mode, it'll pass keys straight to
  underlying (focused) widgets.
- Focus mainly on navigation, and mode switching. Maybe even
  extensibility.
- There's a bunch of functions that won't make sense in every context,
  but will make sense in some contexts. Just leave those open and allow
  other people to easily define them when they need to.
- In piles, keypresses seem to go to the pile first. The keypress is
  sent to the in focus element from there. The in focus element is
  supposed to, from there, handle the key, or return it back so that the
  pile can process it.
- Piles can't handle page up and page down.
- Piles don't seem to allow you to change focus if there are no
	selectables in the pile. At the very least, they don't allow for
	scrolling unless there are some selectables.

# Command Maps

## What has them?

It's looking like selectables have command maps, and you can access them
through the `_command_map` field.

- Buttons
- ListBoxes

## How do They Work?

The command map doesn't call functions. It's just used to determine what
a key is meant to do.

So when defining a keypress function, you could make statements like:

  if command_map[key] == 'command'
	perform command

Or, even better:

  if command_map[key] in [ ...list of possible commands... ]
	{handle said commands}

You can separate mappings from controls using this.

# Good Vim Stuff To Include

- Modes (insert, normal)
  - Optionally visual, for copying the contents of certain boxes.
  - Consider defining a 'yield' for these boxes which controls how
	things come out of the boxes for copying and pasting.
- Numbers before commands
- Maybe ex commands for stuff that I don't want to map keys to, such as
	quitting. Quitting only happens once per run of a program.
- Sequences of keys for commands.

# How to Implement?

## For Keeping It General

Widgets layouts probably won't change on the fly-- at least not the
major container widgets. Syncing up the controls of the wrapper with the
container widget should happen once per program, which isn't so bad.

To link the command map of the wrapper with the command map of the
inside part, I can create a reverse table of the command_map of the
contained widget, then emit whatever key works for the command for the
contained widget.

I'm not sure what I'd do to put new commands inside of a widget which
doesn't have those commands, such as:

- Move to topmost visible (`H`).
- Move to bottommost visible (`L`).
- Move to first widget (`gg`)
- Move to last widget (`G`)

## For Designing Custom Widgets

I'd know exactly which key does what, I wouldn't have to figure out this
puzzle of figuring out which key to transmit to the underlying container
widgets because I'd be using my own container widgets.

## Designing an Interface Instead

Rather than trying to conform to everything possible, I can specify some
things about how to move around, and what functions it can perform for
widgets. Designers of widgets, such as myself, other users of urwid or
the urwid creator could then design their widgets in such a way as to
work with the interface that I specify.

Maybe the widgets can have a method for passing up keybindings.

However, the reverse command map seems to be the most promising method,
currently. The 'interface' specified here could just be specifying the
different kinds of available commands, and what their names would be.
This way, widgets could offer them up in their command maps so that they
could be accessed by the normal mode of the wrapped widget.

That being said, if I create a crapton of navigation things that I'd
want, it might be wasteful to bind keys to these things in the normal
operation. A fair number of widgets might not be able to handle all the
extra keybindings, making their use in insert mode bad. So also consider
creating a way to accept callbacks that will get fired off when a
certain command is given.

# TODO

- Figure out how to deal with key sequences for a command. Is necessary
	for number prefixes to a command.
	- If number, keep waiting, I guess indefinitely.
	- If letter(s), check to see if it's a prefix of more than one valid
		command. If it is, then wait for a *set amount of time* to force
		evaluation of the command string as-is.
- What should it do if it contains no selectable widgets?
- Check the way that urwid widgets tend to handle unhandled keys:
  - When do they call super methods?
	- I think the `super` methods is specifically for when you've
	  inherited from a widget and you want to lean on that widget's
	  functionality when your customizations don't handle it.
  - When do they just return the key?
	- This is probably for when the current widget completely does not
	  handle the key in any way: neither it, nor any class from which it
	  inherits.
- Stretch goals
	- Half page-down
	-	Half page-up
	- Top of page `H`
	- Bottom of page `L`
	- Figure out how to keep normal mode navigation strictly for
		navigating containers, not for moving the cursor around in a box
		(like for Edit).

