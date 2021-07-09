[Objects](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_objects?view=powershell-7.1) in its simplest form are containers that hold items. 
Each Object originates from a .NET base Object. All actions take place within the context of that object.

Each Object contains the following object items:
1. Types - that define the object such as a Collection[],  [String] and [Int].
2. [Event](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/new-event?view=powershell-7.1) –raise a notification about state changes based on conditions such as Network Related, Completion. Status, system conditions 
3. [Methods](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_methods?view=powershell-7.1) – allow you to preform actions on that object instance. 
4. [Properties](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_properties?view=powershell-7.1) – are structured collections that stores relevant object information such as a Number or a String value, etc. 

Get-Member can be used to dump out objects to view them


### Properties used for formatting objects!
[PSTypeNames](https://docs.microsoft.com/en-us/dotnet/api/system.management.automation.pstypename?view=powershellsdk-7.0.0) – Defines the object type. The type links a view.

[PSStandardMembers](https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/extending-properties-for-objects?view=powershell-7.1) – Is a MemberSet. Within this we can define the DefaultDisplayPropertertySet which is a set of properties that should be displayed if there isn’t a view defined.


![image](https://user-images.githubusercontent.com/2629743/125033548-bc5ba580-e08f-11eb-83d0-ccc6a856bfb3.png)
1. You pass a command into the pipeline (Get-Process). A System.Diagnostics.Process object is created
2. This object will pass down the pipeline until it gets to the end of the pipeline. When an object reached the end of the pipeline PowerShell adds  Out-Default to the pipeline (in the background).
3. Out-Default will hand off the object to Out-Host.
4. Out-Host will interact with the PowerShell formatting engine. 

   a. PowerShell formatting system will look up and resolve the object type to see if there is a *predefined* view associated with that object type. (This is stored in the type table cache)
   
   b. IF NOT then the PowerShell formatting system will look to see if there is a user defined view with-in the types.ps1xml and format.ps1xml file (depending on being used within a module or a PowerShell session)
 
   c. IF NOT the formatter will check the object next to see if the DefaultDisplayPropertySet is present. If there is no DefaultDisplayPropertySet 
    NOTE:  For any object in PowerShell, you can access its PSStandardMembers.DefaultDisplayPropertySet property to see the properties that are default for that object, if they are defined
   
   d. IF NOT the PowerShell formatting system will do its best to create a format for the output in a table (default option) or list if there are too many properties to display (over 5 everything goes to a list).


### Extending and formatting objects!
* [TYPES.ps1xml](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_types.ps1xml?view=powershell-7.1) – Extend object types.
  - Properties
  - Methods
  - Scriptblocks
  - …
* [FORMAT.ps1xml](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_format.ps1xml?view=powershell-7.1) – Define how the objects view will be displayed.
  - Table
  - List
  - Wide
  - Custom

* MAML is an XML format used for PowerShell Online help
  Microsoft Assistance Markup Language - [MAML](https://en.wikipedia.org/wiki/Microsoft_Assistance_Markup_Language)
* ELABORATE on this one a little. Types.ps1xml - This file allows you to extend objects with additional properties (aliases, etc.) 
* Format.ps1xml - You have four different views of each object: Table, List, Wide, Custom.
  The formatting affects only the display output. It doesn't affect which object properties are passed down the pipeline or how they're passed.

WARNING: You never want to update these core files because they are signed and can cause damage. What we can do is use one as a template for our custom object.

IMPORTANT: Beginning with PowerShell 6, this information is compiled into PowerShell and is no longer shipped in a Types.ps1xml file.

Files location: Stored in the following location: Get-ChildItem $PSHome/*type*.ps1xml

### How to extend objects!
* Add-Member
* Update-TypeData
* Update-FormatData


## Example demos

### Object Types
* Create a new object with Select-Object

$process = Get-Process | Where-Object {$_.Handles –gt 50} | Sort-Object handles | Select-Object -First 2
$process

* TypeName:	System.Diagnostics.Process

Clear-Host
$process[0] | Get-Member -Force |
Where-Object { ($_.MemberType -eq 'Property') -or ($_.MemberType -eq 'Method') -or ($_.MemberType -eq 'Event') } |
Group-Object MemberType | Format-Table Name, Count

* In-depth view of the object

$process | Get-Member

### PowerShell Object Formatting Hierarchy - Both are hidding properties and can be exposed by using the Get-Member with the -Force parameter

Clear-Host
$process | Get-Member -Force | Where-Object { ($_.Name -eq 'pstypenames') -or ($_.Name -eq 'PSStandardMembers') }  

* Defines the Object Type

$process.pstypenames
https://devblogs.microsoft.com/powershell/psstandardmembers-the-stealth-property/
$process[0].PSStandardMembers.DefaultDisplayPropertySet


### Demo 1
TIP on collection expansion!
Invoke-Item $PSHome\en-us\about_Preference_Variables.help.txt
$FormatEnumerationLimit
$jobs[1].data | Select-Object -First 5 Name, AddressListMembership | Format-Table
$FormatEnumerationLimit = 15
$jobs[1].data | Select-Object -First 5 Name, AddressListMembership | Format-Table
$FormatEnumerationLimit = 4


### Demo 2 - Default-Out and formatting - see above

### Demo 3 
TIP on Select-Object and why not to use it!!! Because it always creates a new object!
$jobs[1].data
$selectedData = $jobs[1].data | Select-Object | Where-Object Name -eq blocked
$selectedData
Look what happened!!!
$selectedData.pstypenames


### Demo 4
PowerShell Pipelining and how PowerShell formats objects
$PSHome = C:\Windows\System32\WindowsPowerShell\v1.0
Clear-Host
Get-ChildItem $PSHome/*format*.ps1xml
Get-ChildItem $PSHome/*type*.ps1xml


### Demo 5
* file types
Notepad $PSHome/dotnettypes.format.ps1xml
Notepad $PSHome/types.ps1xml

* Custom object creation - Need to link to view
NOTE Custom object creation - Need to link to view
Clear-Host
$myCustomObject = [PSCustomObject]@{
    FirstName = 'PowerShell'
    LastName  = 'User'
    Date      = [DateTime]((get-date) -f '%r' -split " ")[0]
    Time      = ((get-date) -f '%r' -split " ")[1]
    City      = 'Charlotte'
    State     = 'NC'
    Job       = 'PowerShell Programmer'
    Company   = 'Software Company'
}

Clear-Host
$myCustomObject | Get-Member
$myCustomObject | Get-Member -Force
$myCustomObject.pstypenames


### Demo 6 
* Ways to extend object with PSTypeNames
 - Method 1 - Create your [PSCustomObject] with PSTypeName = 'Some Object'
$myCustomObject1 = [PSCustomObject]@{
    PSTypeName = 'myCustomObject1'
}
$myCustomObject1.pstypenames

 - Method 2 - Add-Member
$myCustomObject | Add-member -TypeName myCustomObject
Clear-Host
$myCustomObject.pstypenames

 - Method 3 - PSTypeNames.Insert
$myCustomObject3 = [PSCustomObject]@{
    Name = 'Dave'
}
$myCustomObject3.pstypenames.Insert(1, "myCustomObject3")
Clear-Host
$myCustomObject3.pstypenames

NOTE: Add the type: Remove the type: $myCustomobject.pstypenames.Remove('myCustomObject')
Update the property sets to reflect the changes - We will extended the DefaultDisplayProperties
Clear-Host
$myCustomObject
$myCustomObject.pstypenames[0]
Update-TypeData -TypeName myCustomObject -DefaultDisplayPropertySet FirstName, LastName -DefaultDisplayProperty FirstName -DefaultKeyPropertySet CustomProperties -Force
$myCustomObject
$myCustomObject | Get-Member -Force

The data still exists
Clear-Host
"First Name: {0}`nLast Name: {1}`nDate: {2}`nTime: {3}`nCity: {4}`nState: {5}`nJob: {6}`nCompany: {7}" -f $myCustomObject.FirstName,
$myCustomObject.LastName, $myCustomObject.Date, $myCustomObject.Time, $myCustomObject.City,
$myCustomObject.State, $myCustomObject.Job, $myCustomObject.Company


### Demo 7
* Job format creation
Notepad 'c:\temp\MyCustomObject.format.ps1xml'
Update-FormatData -AppendPath 'c:\temp\MyCustomObject.format.ps1xml'
$myCustomObject

* KEY DIFFERENCE between FT and FL (two views exist now!)
$myCustomObject | Format-List
$myCustomObject.PSStandardMembers | Get-Member


### Demo 8 
* Create a new format.ps1xml
Clear-Host
Get-TypeData -TypeName *job*
Get-FormatData -TypeName System.Management.Automation.Job | Export-FormatData -Path c:\temp\jobsxml.ps1xml

* Credit to Bruce Payette for this AWSESOME function!! - Windows PowerShell in Action
Format-XmlDocument -Path C:\temp\jobsxml.ps1xml -BypassEncodingCheck | clip | Notepad

Demo module formats - how it all works
Clear-Host
Get-Job
Get-JobWithFormat

Clear-Host
Get-PSSession
Get-SessionWithFormat
$ses0 = Get-PSSession -id 1
$ses0 | Format-List
$ses1 = Get-SessionWithFormat
$ses1[0] | Format-List
















