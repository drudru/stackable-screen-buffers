# Stackable Screen Buffers

This document is a spec for a new feature in ANSI terminals called 'Stackable Screen Buffers'.

It is like the 'alternate screen buffers' mode that ansi terminals have, except that it doesn't clear the screen. This will allow a new set of command line applications to display dialogs or popup selection UIs.

This mode is created and deallocated by the DEC Private Mode sequence of '2010'.

Table Of Contents
-----------------

* [Quick Example](#section_Quick_Example)
* [Motivation](#section_Motivation)
* [Use Cases](#section_Use_Cases)
* [Specification](#section_Specification)
* [Thoughts](#section_Thoughts)
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

<a name='section_Motivation'></a>
## Motivation

Terminals are not going away. Many computer professionals appreciate the performance they can deliver in modern environments. In fact, we have seen a recent resurgence of interest in using terminals to their fullest specified potential. (Mention taking advantage of modern environments) In fact, new features such as: [Hyperlink Support](https://gist.github.com/egmontkob/eb114294efbcd5adb1944c9f3cb5feda)
(Mention 24 bit color support, unheard of idea back in the 70s)

### Current State

Prior to this feature, a terminal supported two modes. It had a 'normal' mode or an 'alternate screen buffer' mode. We will refer to those modes by those names.

The normal mode is essentially a continuation of the legacy teletype or printer output. Any output here is shown on the screen. Any output that scrolls off the top is typically logged in a history buffer. Applications typically do not use any cursor movement codes in this mode. They will, however, use ANSI SGR codes to set color or font styles to enhance readability.

The 'alternate screen buffer' is the other important mode. When invoked, it preserves the display and then clears it. When this mode is turned off, the previous contents are restored. This mode is typically used by any TUI based applications such as 'less', 'vi/vim', 'emacs', 'top/htop', etc.

The problem with the 'alternate screen buffer' is the clearing of the display. The user loses any contextual information that might have been on the display. (Mention stackoverflow issues).

In todays environment, there is a desire to have small, contextual UI on occassion when in a cli.  However, in all of the modes above, it is not possible for the command line application program to display UI over existing content and then restore it.

### Proposed Solution

What is needed is a new mode that can preserve and restore the state of the terminal. This would give users the ability to have a small overlay on top of their existing screen and not lose context. Since state restoration is the responsibility of the terminal, these programs would be very simple.


<a name='section_Use_Cases'></a>
## Use Cases

### Completion Popups

See [fzf](https://github.com/junegunn/fzf) for an example.

### Dialog Boxes / Windows

Sometimes it is good to see a modal dialog box.

Maybe a mini version of Norton Commander pops up on the top half of your screen.

### Progress Meters

Typical unix programs only show one line of progress output because of the 
limitations mentioned above. At the same time, most processors are multicore
and it is not uncommon to have quite a few parallel jobs running.

You could imagine a programs displaying the progress of an operation using the new mode. Once the 100% mark is reached, it could pop the screen stack and output a status message that will show up on the normal mode display.

### Mini-mux

You could imagine a program that creates a pty and limits the  display output to a subsection of the screen. It creates a buffer, sets the pty size, and then fork/execs a program. Unfortunately, the mini-mux would have to transform ANSI codes that specified on absolute coordinates.

<a name='section_Specification'></a>
## Specification

- The system will interpret a new DEC Private Mode sequence of 2010.
- There is no easy way to detect this modes availability.
- When the mode is set, it will preserve all of the existing terminal state, and create a new overlay with a default alpha of 0.
  - This includes 'normal' or 'alternate screen buffer' modes.
- When the mode is reset, it will destroy the current overlay and restore the previous terminal state.
- If an 'alternate screen buffer' mode is reset, all overlays are destroyed in reverse order and restoring state. Finally, the 'alternate screen buffer' is then destroyed.
- If the terminal doesn't have any buffers on its stack, the terminal ignores the command.
- Terminals will need to support alpha colors. The sequence will be:
set foreground:
```
ESC \[ 38 ; 8 ; <r>; <g>; <b>; <a> m
```
set background:
```
ESC \[ 48 ; 8 ; <r>; <g>; <b>; <a> m
```
The color values are 0 - 255.
- The 'normal' or 'alternate screen buffer' modes will ignore the alpha set on a color.
- When a terminal background color is set to a non 255 alpha, any paints with that background should effectively render all cells below that layer first.
- When a terminal foreground color is set to a non 255 alpha, any paints with that foreground should be done after the last background layer was rendered.
- The terminal should support at least > 1 stackable overlays.

- Some terminal display systems that are focused purely on normal mode display
  should just ignore this mode.
[ansi_up](https://github.com/drudru/ansi_up) for an example.
For example, when viewing the results of a background unix job that displays 
progress (like curl), the 'stackable screen buffer' output should not show up 
in the log.

## Best Practices
<a name='section_Best_Practices'></a>

Any program using this feature will typically need a screen-space rectangle specified.

### Completion

If a shell invoked a completion app, it may provide the app a list of choices on stdin. It should also provide a rectangle argument to the completion app to operate in. The completion app should open '/dev/tty' (the controlling terminal) and send the 'set' sequence. It should then pain in the rect provided. Finally, it should send the 'reset' sequence, and close the '/dev/tty' descriptor.

### Progress

Any program implementing a progress meter should turn on the stackable mode.
At that point, it should display its progress information as usual.
When it completes, it should pop the stackable mode and emit 'normal' mode output to let the user have that status in their terminal history buffer. 


<a name='section_Thoughts'></a>
## Thoughts?

Also, one possible idea is to allow output data to a normal mode display while still leaving the normal mode intact. This would eliminate flicker if switching modes. You could imagine that the display has some parallel job status displaying in the upper screen. At the same time, the system could emit to log messages the normal mode display. These messages would output to the bottom of the display and scroll underneath the stackable buffer. This has problems with multiple stacks, but might be worth it.

We would have to have a sequence that specified output was now going to the 
non-stackable buffer. Old DEC terminals I used supported 'Media Copy'. You could direct output to the AUX port which was typically a serial printer. Maybe we could a sequence similar to that in order to channel output to the normal display for a bit.

## Outstanding Feature Requests

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
