V on operation drop -> the aris rules are not correctly applied live -> the watcher was not getting triggered which means the rerender toggle was not set
	- we already do a slow synchronize so not sure what is holding it back
	- switching out and in edit mode fixes it
- device support
	- depending on device rules you may want to apply different classes
	- select a device rule at the top, anything that is already checked is greyed out, anything else can additionally be checked
		- it is hard (or impossible?) to _uncheck_ something
V the hover effect stays behind once you exit edit mode, need to do a final sweep to remove it!
V cells -> the default variant should be different per alias routed within
	- for instance you may want to define a default variant for all cells containing a form
