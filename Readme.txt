=================
* MP4 Inspector *
=================

-------------
File Contents
-------------

<bin>		Binary files
<source>	Project source codes
Setup.exe	Standalone setup file
License.txt	License agreement
Readme.txt	This file

-------------------
System Requirements
-------------------

.NET Framework 4.0 Client Profile

Microsoft ScriptControl (msscript.ocx) must be installed

-----------------
Version Histories
-----------------

0.0.0.1		Initial beta version

0.0.1.1		Multi-language support (English, Traditional/Simplified Chinese)

0.1.0.1		Macro interface support

0.1.1.1		New members for DataReader object 

0.1.2.0		New setup project (Inno Setup script) 
		Some amendment of DataReader object
		Icon changes for the treeview control

---------------------
About Macro Interface
---------------------

Starting from version 0.1.0.1, macro interface is supported, user can further interpret the box data, or fine tune the data that parsed by the program, whenever the box is clicked by the user, MP4 Inspector will parse that box, and then call the event handler function on macro script (if any), and some related information and data are passed to the function, the function prototype as the following (by VBScript syntax):

Sub OnBoxClick(Path, Version, Flags, DataReader, BoxDataItems)

Path:			The full path of the box is selected
Version:		Version number from the box header
Flags:			Flags number form the box header
DataReader:		A simple stream reader that contains data of the box (box header is excluded)
BoxDataItems:		A data collection of box data to display (Items on ListView control)
BoxDataItem:		An item of BoxDataItems

DatReader Object:

Property Position	The reading position in the stream, increased automatically after Read() is called.
Property Length		The data length (number of bytes), read only.
Property Remainder	Number of bytes left (= Length - Position), read only.
Function Read(Count, [PeekMode] = False)	Read and return bytes array from the stream, argument Count is number of bytes to read, the reading positon will be increased after reading, unless the optional argument PeekMode is set to True.
Function IncPos(Count)	Increase the reading position by integer argument Count (bytes), return value is new reading position.
Function DecPos(Count)	Decrease the reading position by integer argument Count (bytes), return value is new reading position.

BoxDataItems Collection Object:
	
Property Count		Integer number of items in collection, read only.
Property Item(Index)	Default property, return an item, indexed by integer position number.
Function Add(Name, Value)	Add and return a new item to collection, Name is the item title, must be unique in the collection, Value is the data content, both are string type.
Function Find(Name)	Find and return an item by string Name.

BoxDataItem Object:

Property Name		Item title, string value.
Property Value		Item content, string value.

Please note that user interface elements are not allowed to display, when the script is running (message box, etc.), or an error exception will be raised. However, a simple debugger is provided by MP4 Inspector, which can be used to display run time information.

Debugger Object:

Sub Print(Messages)		Print data, Message can be one or multiple (ParemArray).
Sub Assert(Condition, Messages)	Print messages only if Condition is false.
Sub Clear			Clear the existing output data of the debugger.
	
Debugger is located on the simple macro editor, also provided by the program.

Macro Options (Macro editor -> Options):

Enabled			The event handler function will never be called, if the flag set to false.
Scripting Language	VBScript or JavaScript.
Timeout			Maximum running time is allowed for the event handler function.

The script is saved on file "MP4 Inspector.macro" on the folder of the program.

Example (VBScript Version):

Option Explicit

Function Bytes2Str(Bytes)
    Dim I
    For I = 0 To Ubound(Bytes)
        Bytes2Str = Bytes2Str & Chr(Bytes(I))
    Next
End Function

'******************************************************
'Event handler procedure to be called by MP4 Inspector
'******************************************************
Sub OnBoxClick(Path, Version, Flags, DataReader, BoxDataItems)
    Debug.Print Path, FormatNumber(DataReader.Length, 0) 	'Display run-time information
    Select Case Path						'Check to see which box is selected
        Case "\mdat"    
            Dim Data
            DataReader.IncPos 16				'Setup the reading position
            Data = DataReader.Read(8)				'Read bytes from data stream
            BoxDataItems.Add "Custom Item", Bytes2Str(Data)	'Add a new item on the ListView control
        Case "\moov\trak\tkhd"
            Dim Item
            Set Item = BoxDataItems.Find("Duration")		'Find the existing item
            Item.Value = "0x" & Hex(Item.Value)			'Amend the content of the item
    End Select    
End Sub

---------------
Acknowledgement
---------------

Thanks to Bernhard Elbl for the HexBox control.
Thanks to Jordan Russell for the Inno Setup.
