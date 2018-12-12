# How rjsInfraMenu works
The superclass infra.rjsInfraMenu is subclassed in each application library as rjsMenuApp.  This form is used as the main container for all jsClient applications.  This document describes the menu functionality it provides.

There are 5 jsNavMenu objects on the form which provide developer definable menus to present to a user.

NB. Please see the [Omnis Document - Creating Web & Mobile Apps](https://www.omnis.net/download/manuals/Omnis_Webdev81.pdf) Navigation Menu Control on page 170.

1. OrgMenu - displays all organisations that a user has access to in a multi-tenanted application
2. UserMenu - for user specific functions, eg. logout, password change
3. NavMenu - used to navigate between an application's data entry and retrieval forms 
4. NavMenuCascade - identical to NavMenu but in a cascading format for smaller screens, ie. this replaces NavMenu on layout breakpoints 320, 375, 480, 667
5. ToolsMenu - form specific commands plus application wide options

![rjsInfraMenu screenshot](resources/rjsinframenu.png)

Each of the menu objects has an Execute on Client $event with a single line calling its menu click method, eg.

`Do $cinst.$navMenuClick(pLineTag,pLineIdent)`

To avoid confusion, each menu has its own range of numeric ident values:

- NavMenu 1 - 99
- UserMenu 101 - 199
- ToolsMenu 
	- Work 1000 - 1999
	- Notifications 2000 - 2999
	- Search 3000 - 3999
	- Help 4000 - 4999 
- OrgMenu uses the primary key of the entGroupOrganisations record

#### Building the Menus
- user successfully logs in
- rtBusinessApp.$ArrangeAccessList populates task variable tlAuthorisedForms with the forms that the user is permitted to open
- rtSuper.$installMenuForm is called which in turn calls 
- $cinst.$buildNavMenu which calls
- $cinst.$initNavGroups in the subclass rtBusinessApp
- $initNavGroups builds a list of container forms with their menu headings in the order in which they will appear in the NavMenu object and returns the list to $buildNavMenu
- $buildNavMenu then uses tlAuthorisedForms to populate task list variable tlNavMenu with the those items that the user is entitled to see
- $installMenuForm then instantiates rjsMenuApp
- rjsMenuApp copies tlNavMenu to ilNavMenu and ilNavMenuCascade and builds the other menu lists ilToolsMenu, ilUserMenu and, if the app is multi-tenanted and the user can switch between organisations, ilOrgsMenu
- additionally, as the user switches between forms, each form's $construct can call rjsMenuApp.$setupToolsMenu to customise any of the ToolsMenu's drop menus with form specific commands.

#### Using the menus
As mentioned above, each menu has a corresponding $xxxMenuClick method in rjsMenuApp which receives the pLineTag and pLineIdent parameters.

NavMenu and UserMenu - $navMenuClick() executes on server as it access task variables

OrgMenu - $orgMenuClick() also executes on server and clears the current organisation's interface then loads the selected organisation as though the user has just logged in

ToolsMenu - $toolsMenuClick() executes on client using JavaScript to communicate with the currently displayed remote form.  The remote forms have a private execute on client method:

- handleMenuCall - this receives the pLineIdent from the ToolsMenu and calls the appropriate action method.

For example, a simple insert/edit/delete menu will have:

```
;  Return true if this form has handled the shortcut key
Switch pIdent
	Case 1101
		Do $cinst.$insertCR()
		Quit method kTrue
				
	Case 1102
		Do $cinst.$editCR()
		Quit method kTrue 
		
	Case 1103
		Do $cinst.$deleteCR()
		Quit method kTrue
		
	Default
		Quit method kFalse
		
End Switch
```
