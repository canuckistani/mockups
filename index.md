# Mockup of a proposed Terminal

A lot of this is about how the Terminal functions, but it also touches the
larger toolbox, so we'll need to make sure it fits with Shorlanders plans
([Latest?][1])

Here's the basics of what I'm thinking. (Photoshopped)

<img src="screen3.png"/>

If you remember previous iterations:
* The toolbar is gone
* The panel names have changed to 'Terminal' and 'Logging'
* There is a new toolbar button

[1]: http://people.mozilla.com/~shorlander/files/devtools/creation/devtools-creation-i01.html

## Questions

### CoffeeScript from day 1?

Probably not, but we might want it someday. Also there are many other candidates
for compile to JS languages that we could add here. CoffeeScript is just an
initial example.

### How do you switch between JS/Coffee/Commands?

Most people won't need to do this regularly.

There will be a 'lang' command that allows you to change between languages.
Initially the only options will be javascript and command, but we can add
coffee, etc.

Also in javascript mode, you can access commands by prefixing with a ':'.

### Does the terminal include console output?

The terminal will initially include:

* console output both from the page and from the terminal
* uncaught exceptions

The same messages will also be available in the Logging panel, however by
default they will be turned off there. (Are we sure about this?)

If experience shows that we need to, we may add a command to filter the
displayed output:

    : terminal show --console [yes|no|terminal]
    : terminal show --exceptions [yes|no|terminal]

Or something. The exact syntax isn't important unless we're doing this in day 1.

### What about the prompt

We've a number of ideas for prompts. I am currently proposing that we use a
simple > for all languages, and indicate the current language in the icon for
the terminal (see the screenshot above)

We have discussed having the language as part of the prompt either as a symbol
or a name like:

    JS>

However I think this is likely to be extremely repetitive. So long as people
can see the language they are in in the icon they'll be OK. (Perhaps people
should get a reminder in the motd when they first open the terminal)

### What Terminal positions are available?

Initially 2 positions are available

* 'Tab' is what you see.
* 'Below' is attached the the bottom of the toolbox. The Terminal is no longer
  available as a Tab when attached to the bottom
* 'Detached' (probably not day 1)
* 'Side' (probably not day 1) For widescreen monitor use

### What happens to the input area in the console?

It goes away. The console becomes a read-only 'Log' panel.

### What would it look like with Position:Bottom

This is how we'd hack JS while debugging ...

TODO: This needs updating

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
the Terminal? Also the button to toggle page console output is hard to get
right - that's a clumsy name we've got there.

Getting rid of the Position switch:
Could we make do with 2 extra toolbar buttons, one which created a new floating
Terminal (not day 1) and the other which toggled a bottom-attached Terminal.
The tab based Terminal would be toggleable via the options panel still.

### How does completion work?

Our goal with completion is simply to speed the user up. This means:

* Fewer keypresses
* Minimum distraction
* Maximum predictability

Further, it means:

* The user should be able to type 'long-hand' and completion never gets in the
  way.

There are some caveats to that:

* ``<TAB>`` is special. Use ``\t`` (or similar) if you mean the TAB character

This implies:

* ``<LEFT>`` and ``<RIGHT>`` move the cursor rather than accepting completion
* Keys that the user expects to do things should not be stolen by completion
  i.e:

      ``{ window.screen<RETURN>``

  Should do the obvious thing (inspect ``window.screen``) and not nothing.

### What about prefix completion?

Prefix completion is broken. It encourages you to start typing with what is
probably the *most* ambiguous part of what you want to type. Commonly the
right-most part of a parameter is both the least ambiguous and the thing you
are thinking of when you're typing.

Whatever your current directory, if your command-line was smart, it could
probably work out what you meant by:

    : edit gDevTools.jsm

And this is probably true even given several checkouts of central if you have
access to someone's command history.

Conceptually the same is true of JavaScript completion too:

    { href

Probably means ``window.location.href``.

We're away off this kind of command line heaven, but not *that* far off.

So a good model for completion that works however smart your completion system
is is:

* Let the user type, and present them with a best guess (select with ``<TAB>``)
  and some alternatives (pick with ``<UP>``/``<DOWN>`` followed by ``<TAB>``)

### But prefix completion is in my muscle memory

I think we should do everything we can possibly can to help people who have
mental wiring for prefix-completion, in order:

* Ensure that our ``<TAB>`` completions are what people would expect if they
  were clearly thinking of prefix completion. So ...

      ``: edit /Applica<TAB>``

  should complete to ``edit /Applications/``

* Allow a key sequence like ``<SHIFT>+<TAB>`` to do true prefix completion
* If people complain in significant numbers, then offer an option to change
  back

### How are completions presented

When there is a completion that is a suffix of what has been typed so far then
it should be presented in a lighter color inline.

``{ window.setTim|``<span style="color:#99F;">``eout``</span>

(The cursor is the '``|``', and it looks a bit bizarre because Markdown.
Imagine it looks better)

When the completion is not a suffix it should be presented inline in full
following a ``⇥`` (i.e. what's printed on the TAB key)

``{ wndow|`` <span style="color:#99F;">``⇥ window``</span>

When the cursor isn't at the end of the input the same applies

``{ wn|dow`` <span style="color:#99F;">``⇥ window``</span>

The completion proposal should always be appropriate to where the cursor is
however:

``{ wn|dow.setTime`` <span style="color:#99F;">``⇥ window``</span>

``{ wndow.setTime|`` <span style="color:#99F;">``⇥ setTimeout``</span>

Some of this could be hard to code (particularly the last one) so it's OK to
keep things simple by only attempting completion proposals when the cursor is
at the end of the input.

### What if there is more than one completion option?

Options are presented inline, below the input

    { set|Timeout
     | setTimeout   |
     | setInterval  |
     | setResizable |
     '--------------'

I know I'm using ASCII-art above but that's just lazyness. We don't have a
fixation with text-only. This is The Web not some 80s teletype throwback disco.

Having a horizontal menu is nice in that it takes up unused real-estate, but
it's confusing to navigate when ``<LEFT>`` and ``<RIGHT>`` are reserved for the
cursor, so our menus are vertical with a maximum of 5 options.

### What about history?

With an empty command line, ``<UP>`` and ``<DOWN>`` navigate through history.

If the user has started to type and then presses ``<CTRL>+R`` the history is
shown filtered by things that match what has been typed so far

If the user presses ``<CTRL>+R`` to start with then the full history is
initially shown, and is then filtered as the user types.

    { <UP>
     | window.setTimeout    |
     | window               |
     | window.location.href |
     | setTimeout           |
     '----------------------'

    { win<CTRL>+R
     | window.setTimeout    |
     | window               |
     | window.location.href |
     '----------------------'

### What about multi-line input?

Working ...

Multi-line input is tricky because:

* What does ``<RETURN>`` do: new-line or execute? And how does the user discover
  how to do the other thing?
* ``<UP>`` and ``<DOWN>`` can no longer be used for selecting options without
  clashing.

