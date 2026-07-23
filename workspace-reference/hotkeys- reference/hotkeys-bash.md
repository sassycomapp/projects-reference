# Bash

## Cursor movement
- Ctrl+A | Start of line | Moves cursor to the beginning of the current command line.
- Ctrl+E | End of line | Moves cursor to the end of the current command line.
- Ctrl+B | Back one character | Moves cursor one character to the left.
- Ctrl+F | Forward one character | Moves cursor one character to the right.
- Alt+B | Back one word | Moves cursor left to the start of the previous word.
- Alt+F | Forward one word | Moves cursor right to the start of the next word.

## Text deletion / killing
- Ctrl+H | Backspace | Deletes the character to the left of the cursor (same as Backspace).
- Ctrl+D | Delete under cursor | Deletes the character under the cursor (or logs out if line is empty).
- Ctrl+W | Kill previous word | Deletes the word before the cursor and stores it in the kill buffer.
- Alt+D | Kill next word | Deletes the word after the cursor and stores it in the kill buffer.
- Ctrl+K | Kill to end of line | Deletes from cursor to end of line (saved in kill buffer).
- Ctrl+U | Kill to start of line | Deletes from cursor back to start of line (saved in kill buffer).

## Yank / paste / transpose
- Ctrl+Y | Yank (paste) | Pastes the last killed text (from kill buffer) at the cursor.
- Alt+Y | Yank-pop | After Ctrl+Y, cycles through earlier kills to paste different text.
- Ctrl+T | Transpose chars | Swaps the two characters around the cursor.

## History navigation and search
- Up Arrow | Previous command | Steps backward through command history.
- Down Arrow | Next command | Steps forward through command history.
- Ctrl+P | Previous command | Same as Up Arrow; previous history entry.
- Ctrl+N | Next command | Same as Down Arrow; next history entry.
- Ctrl+R | Reverse search history | Incremental search backward through history; type part of a command to find it.
- Ctrl+G | Cancel history search | Exits the current history search without running anything.

## Completion
- Tab | Complete token | Attempts to auto-complete the current word (file/command).
- Shift+Tab | Previous completion (varies) | On some setups, cycles backward through completion options.

## Screen / line control
- Ctrl+L | Clear screen | Clears the terminal display; keeps current command line.
- Ctrl+C | Cancel running command | Sends interrupt signal; stops the current foreground command.
- Ctrl+Z | Suspend command | Pauses current foreground command and returns to shell (can be resumed with fg).
- Ctrl+J | Newline | Equivalent to Enter; executes the current command line.
