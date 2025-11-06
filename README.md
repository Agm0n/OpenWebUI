# (1)
* Added "SystemPromptConfig.svelte" that has a button that opens a dialog letting the user enter a system prompt.
	After the user saves the prompt in the dialog it tells Chat.svelte to use this as "system" in the params (like the admin chat control does).
	(The button is only accessible if files are selected)

* Added values in translation.json (in he-IL) so it will also be in hebrew when hebrew is selected.

# (2)
* Added a way to store avilable knowledgeBases to reduce BE requests ("loadKnowledge" in "src/lib/stores/").

* Changed the BE requests to use this function instead.

* Added a refresh button to force load from the BE when necessary.

* Added values in translation.json (in he-IL) so it will also be in hebrew when hebrew is selected.

# (3)
* Added a new option to the navbar in layout.svelte in the workspace directory for the new "manage chats" window.

* Added a new route with page.svelte on the workspace directory.

* Added the "manage-chats.svelte" window.
	- In that window I made a table for displaying every chat and its values.
	- Made it so that when clicking it will direct the user to the selected chat.
	- Added tags.
	- Made it order based on the selected column.
	- Added an edit button that opens a dialog for changing the name and tags of the selected chat.

* Added values in translation.json (in he-IL) so it will also be in hebrew when hebrew is selected.
