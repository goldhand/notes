My Notes
========
My notes on things.


Notes Index
-----------
* [dump remote db](/notes/dump_remote_db.md)
* [eb setting up a new instance](/notes/eb_setting_up_a_new_instance.md)
* [es6 maps and weakmaps](/notes/es6_maps_and_weakmaps.md)
* [js data types](/notes/js_data_types.md)
* [progressive web apps](/notes/progressive_web_apps.md)
* [react prop types](/notes/react_prop_types.md)
* [redux adding a reducer](/notes/redux_adding_a_reducer.md)
* [redux middleware](/notes/redux_middleware.md)
* [service workers](/notes/service_workers.md)
* [configuring AVA for testing with a react / webpack project](/notes/testing_react_webpack_with_ava_and_enzyme.md)
* [javascript error handling](/notes/javascript_error_handling.md)



Note Taking in Terminal
-----------------------
Using this tool for taking notes from your terminal.

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
