# Stackable Screen Buffers

This document is a spec for a new feature in ANSI terminals called 'Stackable Screen Buffers'.

It is like the 'alternate screen buffers' mode that ansi terminals have, except that it doesn't clear the screen. This will allow a new set of command line applications to display dialogs or popup selection UIs.

This mode is created and deallocated by the DEC Private Mode sequence of '2010'.

Table Of Contents
-----------------

* [Quick Example](#section_Quick_Example)
* [Use Cases](#section_Use_Cases)
* [Specification](#section_Specification)
* [Feature Requests](#section_Feature_Requests)


<a name='section_Quick_Example'></a>
## Quick Example

Here's a simple command to try out the feature. The result is equivalent to this HTML link: [This is a link](http://example.com)

To push a new buffer:
```
echo -e '\e]?2010h'
```
To pop a new buffer:
```
echo -e '\e]?2010l'
```

<a name='section_Use_Cases'></a>
## Use Cases

Terminals are not going away. People appreciate the performance they can have in modern environments. In fact, we have seen a recent resurgence of interest in using terminals to their fullest specified potential. (Mention taking advantage of modern environments) In fact, new features such as: [Hyperlink Support](https://gist.github.com/egmontkob/eb114294efbcd5adb1944c9f3cb5feda)

### Current State
Prior to this feature, a terminal supported two modes. It had a 'normal' mode or an 'alternate screen buffer' mode. The normal mode is essentially a continuation of the legacy teletype or printer output. Any output here is logged in a history buffer. Applications typically do not use any cursor movement codes in this mode. They will, however, use ANSI SGR codes to set color or font styles to enhance readability.

The 'alternate screen buffer' is the other important mode. When invoked, it preserves the display and then clears it. When this mode is turned off, the previous contents are restored. This mode is typically used by any TUI based applications such as 'less', 'vi/vim', 'emacs', 'top/htop', etc.

The problem with the 'alternate screen buffer' is the clearing of the display. The user loses any contextual information that might have been on the display.
the other important mode. When invoked, it preserves the display and then clears it. When this mode is turned off, the previous contents are restored. This mode is typically used by any TUI based applications such as 'less', 'vi/vim', 'emacs', 'top/htop', 'midnight commander', 'ranger', etc.

In all of the modes above, it is not possible for the command line application program to read the contents of the screen. Given that constraint, and the above modes.

One workaround is to 


<a name='section_Use_Cases'></a>
## Use Cases

### Completion Popups

### Dialog Boxes

### Progress Meters

Displaying progress.
You could imagine a programs displaying the progress of an operation using the new mode. Once the 100% mark is reached, it could pop the screen stack and output a status message that will show up on the normal mode display.

### Mini-mux

You could imagine a program that creates a pty and limits the  display output to a subsection of the screen.

<a name='section_Specification'></a>
## Specification

- The system will interpret a new DEC Private Mode sequence of 2010.
- When the mode is set, it will preserve all of the existing terminal, and create a new overlay with an alpha of 0.
- When the mode is reset, it will destroy the current overlay and restore the previous terminal state.
- If the terminal doesn't have any buffers on its stack, the terminal ignores the command.
- When a terminal SGR of is set, it should effectively treat those set characters as alpha of those characters to 0. Any lower buffers, should show through to the upper layer. 
- There is no easy way to detect this mode.

<a name='section_Feature_Requests'></a>
## Feature Requests

- [tmux](https://github.com/tmux/tmux/issues/xxx)
- [fzf](https://github.com/junegunn/fzf)

```
╔═ file1 ════╗
║          ╔═ file2 ═══╗
║http://exa║Lorem ipsum║
║le.com    ║ dolor sit ║
║          ║amet, conse║
╚══════════║ctetur adip║
           ╚═══════════╝
```
