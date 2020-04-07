.

# Several tooling and scripts I have created

* [Autodesk Revit Add-In](#basic-bim-checker-for-autodesk-revit)<br>
* [Autodesk Navisworks Manage Tool](#basic-bim-checker-for-navisworks-manage)<br>
* [Creating Smart Views for BimCollabZoom](#creating-xml-smart-views-for-bimcollabzoom)<br>
* Creating QR codes from an IFC file<br>
* Using IFC as a Document Management System<br>
* Creating a folder structure from an Excel classification file<br>
* Getting and validing Quantities from an IFC file<br>

-------

# Basic-BIM-Checker-for-Autodesk-Revit
[< Back to top](#several-tooling-and-scripts-i-have-created)

A Revit-Add in which the user can make a visual check if the model is according the Information Delivery Manual

[Basic BIM Checker for Revit](https://github.com/C-Claus/Basic-BIM-Checker-for-Autodesk-Revit/blob/master/README.md)

![Revit Add-In](/images/Addln.png)

-------

# Basic-BIM-Checker-for-Navisworks-Manage
[< Back to top](#several-tooling-and-scripts-i-have-created)

A stand-alone Navisworks tool which makes Search Sets according to the Information Delivery Manual

[Basic BIM Checker for Navisworks](https://github.com/C-Claus/Basic-BIM-Checker-for-Autodesk-Navisworks-Manage/blob/master/README.md)

![Revit Add-In Navis](/images/nav_app.png)

-------

# Creating-XML-Smart-Views-for-BimCollabZoom
[< Back to top](#several-tooling-and-scripts-i-have-created)

BimCollabZoom is software to view and coordinate IFC files. In my opinion it's the most complete and easy-to-use IFC viewer from all IFC viewers I know. Reasons for this is that the user is allowed to create Smart Views and BCF files for free. The Smart Views are XML files which gave me an idea. I can use the lxml Python module to create and modify the Smart Views. Especially with large repetive buildings tedious task can become interesting challenges and at the same time you can thoroughly check the IFC model for complete and correctness.

![spaces](/images/screenshot_bimcollabzoom.png)
Screenshot of BimCollabZoom 



![spaces](/images/spaces.PNG)
The IfcSpaces visualized, not that it is hard to see which IfcSpace are contained within the IfcZone. And on what IfcBuildingStorey they are located.



![smart_view](/images/smart_view.png)
The IfcSpaces and IfcZone visualized with color coding, the Smart View has been created with a Python script.

```
import ifcopenshell
import lxml 
from lxml import etree as ET
import os 
import uuid
from datetime import datetime
from collections import defaultdict
```
The imports used to create the Smart Views

```
ifcfile = ifcopenshell.open('C:\\Users\\CClaus\\Desktop\\Flat 11\\ruimtemodel_flat_11.ifc')

products = ifcfile.by_type('IfcProduct')
zones = ifcfile.by_type('IfcZone')
building_stories = ifcfile.by_type('IfcBuildingStorey')
```
Using IfcOpenShell to retrieve the information in IFC to create the data written to the XML file.


```
def create_xml(file_name, building_storey, zone):
  
    root = ET.Element('SMARTVIEWSETS')
    doc = ET.SubElement(root, "SMARTVIEWSET")
 
    title = ET.SubElement(doc, "TITLE").text = "location_" + building_storey
    description = ET.SubElement(doc, "DESCRIPTION").text = "Description will follow" 
    guid = ET.SubElement(doc, "GUID" ).text = str(uuid.uuid4())
    smartviews = ET.SubElement(doc, "SMARTVIEWS")
    
```
Using the lxml module to initialize the xml within a method.
