
# Mockup of a proposed REPL

Here's the basics of what I'm thinking. (Photoshopped)

<img src="screen.png"/>

## Questions

### CoffeeScript from day 1?

Maybe not, but we might want it someday. Also there are many other candidates
for compile to JS languages that we could add here. CoffeeScript is just an
initial example.

### How do you switch between JS/Coffee/Commands?

You can click the buttons in the toolbar or press magic keys.

I expect the buttons might read like this:

      [ JavaScript { | CoffeeScript ; | Command line : ]

To give users the clue that you can also switch by pressing one of { ; or :
And yes, I made the shortcut for CoffeeScript ';'. I may relent.

When you execute a JS command, we stay in JS mode, same for CoffeeScript and
Commands.

Also: Under consideration - change the prompt from '>' to reflect the current
mode [{|;|:]

### Does the REPL include console output?

There are 3 options:

1. No, never
2. Yes, always
3. Yes if the console.xxx command was typed in the REPL, not otherwise

The 'Console' dropdown allows toggling between these 3.
The default is 3. We're trying to draw a line between being clean and not
confusing the heck out of people who type `console.log(foo);` and then can't
work out where the output went.

The Log panel includes console output from the REPL and the page, and allows
for filtering by level.

### What REPL positions are available?

2 or 3 or 4.
In order of importance:

* 'Tab' is what you see.
* 'Below' is attached the the bottom of the toolbox. The REPL is no longer
  available as a Tab when attached to the bottom
* 'Detached' (probably not day 1)
* 'Side' (probably not day 1) For widescreen monitor use

### What happens to the input area in the console?

It goes away. The console becomes a read-only 'Log' panel.

### What would it look like with Position:Bottom

This is how we'd hack JS while debugging ...

<img src="screen2.png"/>

I forgot to update the text in the "Position" drop-down. Shoot me.

### To consider

Could we get rid of the toolbar?
The motivation for doing this: It keeps things clean and options are often just
a crutch for developers failing to make a decision.

Getting rid of the 'language' switch:
The switch wouldn't be used that often, we could add some usage hints to the
startup page.
OTOH I think we need to be ultra-clear about the language that is being spoken
and knowing how to switch is important too.

Getting rid the 'Page Console Output' switch:
Actually how much mess would we create by just adding *all* console output to
the REPL? Also the button to toggle page console output is hard to get right -
that's a clumsy name we've got there.

Getting rid of the Position switch:
Could we make do with 2 extra toolbar buttons, one which created a new floating
REPL (not day 1) and the other which toggled a bottom-attached REPL. The tab
based REPL would be toggleable via the options panel still.

### How does completion work?

That's next.
