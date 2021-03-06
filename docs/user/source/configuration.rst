*************
Configuration
*************


All configs are optional, but if you want to send mails you need to
specify at least one account section.

Alot reads a config file in the "INI" syntax:
It consists of some sections whose names are given in square brackets, followed by
key-value pairs that use "=" or ":" as separator, ';' and '#' are comment-prefixes.

The default location for the config file is `~/.config/alot/config`.
You can find a complete example config with the default values and their decriptions in
`alot/defaults/alot.rc`.

Note that since ":" is a separator for key-value pairs you need to use "colon" to bind
commands to ":".

Here is a key for the interpreted sections:

[general]
    global settings: set your editor etc
[account X]
    defines properties of account X: (see below)
[X-maps]
    defines keymaps for mode X. possible modes are:
    envelope, search, thread, taglist, bufferlist and global.
    global-maps are valid in all modes.
[tag-translate]
    defines a map from tagnames to strings that is used when
    displaying tags. utf-8 symbols welcome.
[Xc-theme]
    define colour palette for colour mode. X is in {1, 16, 256}.


Accounts
========
A sample gmail section looks like this (provided you have configured msmtp accordingly)::

    [account gmail]
    realname = Patrick Totzke
    address = patricktotzke@gmail.com
    aliases = patricktotzke@googlemail.com
    sendmail_command = msmtp --account=gmail -t

Here's a full list of the interpreted keywords in account sections::

    # used to format the (proposed) From-header in outgoing mails
    realname = your name
    address = this accounts email address

    # used to clear your addresses/ match account when formating replies
    aliases = foobar@myuni.uk;f.bar@myuni.uk;f.b100@students.myuni.uk

    # how to send mails
    sendmail_command = command, defaults to 'sendmail'

    # where to store outgoing mail
    sent_box = maildir:///home/you/mail//Sent

    # how to tag sent mails [default: sent]. seperate multiple tags with ','.
    sent_tags = sent

    # path to signature file
    signature = ~/your_vcard_for_this_account.vcs

    # attach signature file if set to True, append its content (mimetype text)
    # to the body text if set to False. Defaults to False.
    signature_as_attachment = False

    # signature file's name as it appears in outgoing mails if
    # signature_as_attachment is set to True
    signature_filename = you.vcs

    # command to lookup contacts
    abook_command = abook --mutt-query
    abook_regexp = regexp to match name & address in abook_commands output.

.. warning::

  Sending mails is only supported via sendmail for now. If you want
  to use a sendmail command different from `sendmail`, specify it as `sendmail_command`.

`send_box` specifies the mailbox where you want outgoing mails to be stored
after successfully sending them. You can use mbox, maildir, mh, babyl and mmdf
in the protocol part of the url.

The file specified by `signature` is attached to all outgoing mails from this account, optionally
renamed to `signature_filename`.

If you specified `abook_command`, it will be used for tab completion in queries (to/from)
and in message composition. The command will be called with your prefix as only argument
and its output is searched for name-email pairs. The regular expression used here
defaults to `(?P<email>.+?@.+?)\s+(?P<name>.+)`, which makes it work nicely with `abook --mutt-query`.
You can tune this using the `abook_regexp` option (beware Commandparsers escaping semantic!).
Have a look at the FAQ for other examples.


Key Bindings
============
If you want to bind a commandline to a key you can do so by adding the pair to the
`[MODE-maps]` config section, where MODE is the buffer mode you want the binding to hold.
Consider the following lines, which allow you to send mails in envelope buffers using the
combination `control` + `s`::

    [envelope-maps]
    ctrl s = send

Possible MODE strings are:

* envelope
* search
* thread
* taglist
* bufferlist
* global

Bindings defined in section `[global-maps]` are valid in all modes.

Have a look at `the urwid User Input documentation <http://excess.org/urwid/wiki/UserInput>`_ on how key strings are formated.



Hooks
=====
Hooks are python callables that live in a module specified by
`hooksfile` in the `[global]` section of your config. Per default this points
to `~/.config/alot/hooks.py`.
For every command X, the callable 'pre_X' will be called before X and 'post_X' afterwards.

When a hook gets called, it receives instances of

ui
  `alot.ui.UI`, the main user interface object that can prompt etc.
dbm
  `alot.db.DBManager`, the applications database manager
aman
  `alot.account.AccountManager`, can be used to look up account info
config
  `alot.settings.config`, a configparser to access the users config

An autogenerated API doc for these can be found at http://alot.rtfd.org ,
the sphinx sources live in the `docs` folder.
As an example, consider this pre-hook for the exit command,
that logs a personalized goodby message::

    import logging
    def pre_exit(aman=None, **rest):
        accounts = aman.get_accounts()
        if accounts:
            logging.info('goodbye, %s!' % accounts[0].realname)
        else:
            logging.info('goodbye!')

Apart from command pre and posthooks, the following hooks will be interpreted:

`reply_prefix(realname, address, timestamp, **kwargs)`
    Is used to reformat the first indented line in a reply message.
    Should return a string and defaults to 'Quoting %s (%s)\n' % (realname, timestamp)
`forward_prefix(realname, address, timestamp, **kwargs)`
    Is used to reformat the first indented line in a inline forwarded message.
    Returns a string and defaults to 'Forwarded message from %s (%s)\n' % (realname, timestamp)
`pre_edit_translate(bodytext, **kwargs)`
    can be used to manipulate a messages bodytext before the editor is called.
    Receives and returns a string.
`post_edit_translate(bodytext, **kwargs)`
    can be used to manipulate a messages bodytext after the editor is called
    Receives and returns a string.

    

Widget Colours
==============
Alot can be run in 1, 16 or 256 colour mode.
The requested mode is determined by the commandline parameter `-C` or read from
option `colourmode` in section `[globals]` of your config file.
The default is 256, which will be scaled down depending on how many colours
your terminal supports.

The interface will theme its widgets according to the palette defined in
section `[MODEc-theme]` where `MODE` is the integer indicating the colour mode.
Have a look at the default config (`alot/defaults/alot.rc`) for a complete list
of interpreted widget settings; the keys in this section should be self-explanatory.

Values can be colour names (`light red`, `dark green`..), RGB colour codes (e.g. `#868`),
font attributes (`bold`, `underline`, `blink`, `standout`) or a comma separated combination of
colour and font attributes.
In sections `[16c-theme]` and `[256c-theme]` you can define Y_fg and
Y_bg for the foreground and background of each widget keyword Y, whereas the monochromatic
(`[1c-theme]`) palette can only interpret font attributes for key Y without the suffix.
As an example, check the setting below that makes the footer line appear as
underlined bold red text on a bright green background::

    [256c-theme]
    global_footer_bg = #8f6
    global_footer_fg = light red, bold, underline

See `urwids docs on Attributes <http://excess.org/urwid/reference.html#AttrSpec>`_ for more details
on the interpreted values. Urwid provides a `neat colour picker script`_ that makes choosing
colours easy.

.. _neat colour picker script: http://excess.org/urwid/browser/palette_test.py


Custom Tagstring Formatting
===========================
In theme sections you can use keys with prefix `tag_` to format specific tagstrings. For instance,
the following will make alot display the "todo" tag in white on red when in 256c-mode. ::

    [256c-theme]
    tag_todo_bg = #d66
    tag_todo_fg = white

You can translate tag strings before displaying them using the `[tag-translate]` section. A
key=value statement in this section is interpreted as:
Always display the tag `key` as string `value`. Utf-8 symbols are welcome here, see e.g.
http://panmental.de/symbols/info.htm for some fancy symbols. I personally display my maildir flags
like this::

    [tag-translate]
    flagged = ⚑
    unread = ✉
    replied = ⇄

Highlighting Search Results
===========================
Thread lines in the ``SearchBuffer`` can be highlighted by applying a theme different
from their regular one if they match a `notmuch` query.

The default config predefines highlighting for threads that carry the `unread`,
the `flagged` or both of those tags.

Thread lines consist of up to six components (not all of which are shown by
default) that may be themed individually to provide highlighting. The components
are 

 - `date`
 - `mailcount`
 - `tags`
 - `authors`
 - `subject`
 - `content`
 
Have a look at Alot's interface to see what they are.

Customizing highlighting, you may define which components you want highlighted.
Add a `highlighting` section to your config file and define a comma separated
list of highlightable components: ::

    [highlighting]
    components = date, mailcount, tags, authors, subject

Rules
-----
To specify which threads should be highlighted, you need to define highlighting
rules. Rules map queries onto theme identifiers. Each thread that matches a given rule
will use a theme identified by the ID the rule is mapped to.

.. admonition:: Example

    To highlight threads that are tagged as 'important', add the `rules`
    key to your `highlighting` section and provide a dict in JSON syntax. Use an
    appropriate `notmuch` query as a key and select a meaningful theme identifier as
    its value:
    
::

    rules = { "tag:important":"isimportant" }

.. note::
  Please make sure the identifier isn't the name of an actual tag, since this
  may introduce ambiguity when highlighting tags. More on that `later`_.

If you want highlighting for other threads as well, just add more rules to the
dict: ::

    rules = { "tag:important":"isimportant",
              "subject:alot":"concernsalot",
              "from:mom@example.com":"frommom"}

.. note:: 
    The sequence of the list defines the search order. The first query that
    matches selects the highlighting. So if you have queries that are harder to
    satisfy, you should put them earlier in the dict than ones that match more
    easily:

::

    rules = { "tag:unread":"isunread",
              "tag:unread AND tag:important":"isunreadimportant"}

This setup will never highlight any threads as `isunreadimportant`, since alle
threads that would match that identifier's query will *also* have matched the
`isunread` query earlier in the rules dict. So, again, make sure that rules that
are hard to satisfy show up early in the dict: ::

    rules = { "tag:unread AND tag:important":"isunreadimportant",
              "tag:unread":"isunread"}

This way only threads that didn't match `isunreadimportant` before end up
highlighted as `isunread` only.

.. _later: `ambiguous theme identifiers`_

Theme Generic Components
------------------------
.. note:: 
  The following schema will allow you to define highlighting themes for all
  components *except* `tags`, which follow a different system and will be
  explained in the `next section`_.

To define a highlighting theme for a component, you need to add a key of the
following format to your colour theme (please cf. `Widget Colours`_ for more information
on theming): ::

   search_thread_COMPONENT_ID_[focus_][fg|bg]

where 

 - ``COMPONENT`` is the component this theme is meant to highlight,
 - ``ID`` is the theme identifier that defines which query this option belongs
   to,
 - ``focus_`` is optional and if present defines that the theme should only be
   used if the current thread is focussed and
 - ``fg`` or ``bg`` is a selection that specifies which themable part of the
   component this option refers to.

.. admonition:: Example

    The following option will highlight the `subject` of each thread that
    matches the query mapping to `isimportant` if the current thread is
    `focus`\sed by theming its `foreground` according to the values stated
    below:

::
    
    search_thread_subject_isimportant_focus_fg = dark red, underline

Following this pattern will allow you to set theming for the `background`, for
the `subject` of threads tagged as `important` that are currently not focussed
(by omitting the `focus_` part of the key string), for `subject`\s of threads
matching a different query, and all other components except `tags`.

.. _next section: `Theme Tags Component`_

Theme `Tags` Component
----------------------
As described in `Custom Tagstring Formatting`_, tags may be themed individually.
Highlighting expands this concept by allowing default themed tags as well as
individual themed tags to provide highlighting variants.

To specify highlighting themes for default themed tags, just add a key with the wanted
theme identifier: ::

    tag_ID_[focus_][fg|bg]

where

 - ``ID`` is the theme identifier that defines which query this option belongs
   to,
 - ``focus_`` is optional and if present defines that the theme should only be
   used if the current thread is focussed and
 - ``fg`` or ``bg`` is a selection that specifies which themable part of the
   component this option refers to.

To highlight custom themed tags, proceed accordingly. Specify ::

   tag_TAG_ID_[focus_][fg|bg]

where

 - ``TAG`` is the name of the custom themed tag that is to be highlighted,
 - ``ID`` is the theme identifier that defines which query this option belongs
   to,
 - ``focus_`` is optional and if present defines that the theme should only be
   used if the current thread is focussed and
 - ``fg`` or ``bg`` is a selection that specifies which themable part of the
   component this option refers to.

.. _ambiguous theme identifiers:
.. caution::
    As mentioned earlier, using tag names as theme identifiers may introduce
    ambiguity and lead to unexpected theming results. 

Assuming one would replace the theme identifier `isimportant` with its intuitive
alternative `important`, the tag theme ``tag_important_fg`` might either be a
custom theme for the tag `important` of the form ``tag_TAG_fg`` or the highlight
theme for default themed tags of threads that match the query that maps to the
`important` identifier: ``tag_ID_fg``.

Using above proper identifier would distinguish those options as
``tag_important_fg`` for the custom theme and ``tag_isimportant_fg`` for the
highlighting theme.


Contacts Completion
===================
In each `account` section you can specify a `abook_command` that
is considered the address book of that account and will be used
for address completion where appropriate.

This shell command will be called with the search prefix as only argument.
Its output is searched for email-name pairs using the regular expression given as `abook_regexp`,
which must include named groups "email" and "name" to match the email address and realname parts
respectively. See below for an example that uses `abook <http://abook.sourceforge.net/>`_::

    [account YOURACCOUNT]
    realname = ...
    address = ...
    abook_command = abook --mutt-query
    abook_regexp = (?P<email>.+?@.+?)\s+(?P<name>.+)

See `here <http://notmuchmail.org/emacstips/#index11h2>`_ for alternative lookup commands. The few others I have tested so far are:

`goobook <http://code.google.com/p/goobook/>`_
    for cached google contacts lookups::

      abook_command = goobook query
      abook_regexp = (?P<email>.+?@.+?)\s\s+(?P<name>.+)\s\s+.+

`nottoomuch-addresses <http://www.iki.fi/too/nottoomuch/nottoomuch-addresses/>`_
    completes contacts found in the notmuch index::

      abook_command = nottoomuch-addresses.sh
      abook_regexp = \"(?P<name>.+)\"\s*<(?P<email>.*.+?@.+?)>

Don't hesitate to send me your custom `abook_regexp` values to list them here.
