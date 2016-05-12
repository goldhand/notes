Note Taking in Terminal
=======================
Simple tool for taking notes from your terminal.

Usage
-----
To take a new note:

    $ notes -a <note_name>

To review a note:

    $ notes  # you will be prompted with a list of notes

To edit a note:

    $ notes -e  # you will be prompted with a list of notes


Setup
-----
Copy this `notes` function from this [dotfile](https://github.com/goldhand/dotfiles/blob/master/.functions#L283) into your `.bashrc` or whatever `.bash` dotfile you want as long as it gets sourced when your terminal starts a new session.

Also add two exports to your `.bashrc` somewhere before the `notes` function:

    # Path for notes
    export NOTES_DIR=$HOME/notes/notes

    # Text editor for notes
    export NOTES_EDITOR=vim

You can change the directory and editor to whatever you want. Just make sure the directory exists and the editor has an alias.
