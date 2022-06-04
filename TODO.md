!IMPORTANT
breakpoints do not take into account fixed-width containers
for instance if you have 1024px wide container, the content will still change at the breakpoints
usually for smaller containers this is not too bad as you actually end up with less space anyway
HOWEVER, for the widescreen this _is_ a problem. even if you go wide, if you cap it at 1024px, you will suddenly get more items on a row
need something to prevent this!


- COMPONENT STATE
	currently only known to the page _after_ the component loads, depending on other components that have a dependency, this might be too late
	on save (of a page), we should list all "named" components and add them to the page variables
	so they can be initialized at startup
	when anyone asks, return it from variables, not from the target component, it is up to the component to make sure everything is reactive

- content-width is not yet responsive!
V on operation drop -> the aris rules are not correctly applied live -> the watcher was not getting triggered which means the rerender toggle was not set
	- we already do a slow synchronize so not sure what is holding it back
	- switching out and in edit mode fixes it
V device support: properly added using column based approach
	- depending on device rules you may want to apply different classes
	- select a device rule at the top, anything that is already checked is greyed out, anything else can additionally be checked
		- it is hard (or impossible?) to _uncheck_ something
V the hover effect stays behind once you exit edit mode, need to do a final sweep to remove it!
V cells -> the default variant should be different per alias routed within
	- for instance you may want to define a default variant for all cells containing a form
