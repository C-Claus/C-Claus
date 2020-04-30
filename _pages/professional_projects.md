.

# Several tooling and scripts I have created

* [Autodesk Revit Add-In](#basic-bim-checker-for-autodesk-revit)<br>
* [Autodesk Navisworks Manage Tool](#basic-bim-checker-for-navisworks-manage)<br>
* [Creating Smart Views for BimCollabZoom](#creating-xml-smart-views-for-bimcollabzoom)<br>
* [Adding properties to an IFC file](#adding-properties-to-an-ifc-file)<br>
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

BimCollabZoom is software to view and coordinate IFC files. In my opinion it's the most complete and easy-to-use IFC viewer from all IFC viewers I know. Reasons for this is that the user is allowed to create Smart Views and BCF files for free. The Smart Views are XML files which gave me an idea. I can use the lxml Python module to create and modify the Smart Views. Especially with large repetive buildings tedious task can become interesting challenges. At the same time you can thoroughly check the IFC model for complete- and correctness.

![spaces](/images/screenshot_bimcollabzoom.png)
Screenshot of BimCollabZoom 



![spaces](/images/spaces.PNG)
The IfcSpaces visualized, not that it is hard to see which IfcSpace are contained within the IfcZone. And on what IfcBuildingStorey they are located.



![smart_view](/images/smart_view.png)
The IfcSpaces and IfcZone visualized with color coding, the Smart View has been created with a Python script.

```
import os 
import lxml 
import uuid
import ifcopenshell

from lxml import etree as ET
from datetime import datetime
from collections import defaultdict

```
The imports used to create the Smart Views

```
ifcfile = ifcopenshell.open('C:\\Users\\CClaus\\Desktop\\Flat 11\\ruimtemodel_flat_11.ifc')

products = ifcfile.by_type('IfcProduct')
zones = ifcfile.by_type('IfcZone')
building_stories = ifcfile.by_type('IfcBuildingStorey')

building_storey_list = []

for i in building_stories:
    building_storey_list.append(i.Name)

now = datetime.now()
year = now.strftime("%Y")
month = now.strftime("%m")
day = now.strftime("%d")
```
Using IfcOpenShell to retrieve the information in IFC to create the data written to the XML file. Including the initialisation of some global variables.



    def create_xml(file_name, building_storey, zone):
        root = ET.Element('SMARTVIEWSETS')
        doc = ET.SubElement(root, "SMARTVIEWSET")
 
        title = ET.SubElement(doc, "TITLE").text = "location_" + building_storey
        description = ET.SubElement(doc, "DESCRIPTION").text = "Description will follow" 
        guid = ET.SubElement(doc, "GUID" ).text = str(uuid.uuid4())
        smartviews = ET.SubElement(doc, "SMARTVIEWS")
    

Using the lxml module to initialize the xml within a method.

```
    for i in zone:
        smartview = ET.SubElement(smartviews, "SMARTVIEW")
        smartview_title = ET.SubElement(smartview, "TITLE").text = 'Building number: ' + str(i) #+ str(i.Name) 
        smartview_description = ET.SubElement(smartview, "DESCRIPTION")
        smartview_creator = ET.SubElement(smartview, "CREATOR").text = "C. Claus"
        smartview_creation_date = ET.SubElement(smartview, "CREATIONDATE").text = day + " " + month + " " + year
        smartview_modifier = ET.SubElement(smartview, "MODIFIER").text = "C. Claus"
        smartview_modification_date = ET.SubElement(smartview, "MODIFICATIONDATE").text = day + " " + month + " " + year
        smartview_guid = ET.SubElement(smartview, "GUID").text = str(uuid.uuid4())
        
        first_rule(smartview, rules,  building_storey)
        
        second_rule(smartview, rules, zones=i)
        
        third_rule(smartview, rules, building_storey='building storey')
        
    tree = ET.ElementTree(root)
    tree.write('bcsv_files/' + str(file_name), encoding="utf-8", xml_declaration=True, pretty_print=True)
   
    create_xml_declaration(file_path_xml='bcsv_files/' + str(file_name))
 ```    
 
 Looping over the zone within to create the names of the SmartViews.

```
def first_rule(smartview, rules, building_storey): 
    
        rule = ET.SubElement(rules, "RULE")

        ifctype = ET.SubElement(rule, "IFCTYPE").text = 'Building Story'
        
        property = ET.SubElement(rule, "PROPERTY")
        property_name = ET.SubElement(property, "NAME").text= "Name"
        propertyset_name = ET.SubElement(property, "PROPERTYSETNAME").text = "Summary"
        property_type = ET.SubElement(property, "TYPE").text = "Summary"
        property_value_type = ET.SubElement(property, "VALUETYPE").text = "StringValue"
        property_unit = ET.SubElement(property, "UNIT").text = "None"
        
        
        condition = ET.SubElement(rule, "CONDITION")
        condition_type = ET.SubElement(condition, "TYPE").text = "StartsWith"
        condition_value = ET.SubElement(condition, "VALUE").text =  building_storey
        
        action = ET.SubElement(rule, "ACTION")
        action_type = ET.SubElement(action, "TYPE").text = "AddSetColored"
        
        r_color = ET.SubElement(action, "R").text = "145"
        g_color = ET.SubElement(action, "G").text = "145"
        b_color = ET.SubElement(action, "B").text = "145"
```
Creating a method for the first rule, this one makes a Smart View for the Building Storey.



```
def second_rule(smartview, rules, zones):     
    
        #rules = ET.SubElement(smartview, "RULES")
        rule = ET.SubElement(rules, "RULE")

        ifctype = ET.SubElement(rule, "IFCTYPE").text = 'Zone'
        
        property = ET.SubElement(rule, "PROPERTY")
        property_name = ET.SubElement(property, "NAME").text= "Name"
        propertyset_name = ET.SubElement(property, "PROPERTYSETNAME").text = "Summary"
        property_type = ET.SubElement(property, "TYPE").text = "Summary"
        property_value_type = ET.SubElement(property, "VALUETYPE").text = "StringValue"
        property_unit = ET.SubElement(property, "UNIT").text = "None"
        
        
        condition = ET.SubElement(rule, "CONDITION")
        condition_type = ET.SubElement(condition, "TYPE").text = "StartsWith"
        condition_value = ET.SubElement(condition, "VALUE").text =  zones
        
        action = ET.SubElement(rule, "ACTION")
        action_type = ET.SubElement(action, "TYPE").text = "AddSetColored"
        
        r_color = ET.SubElement(action, "R").text = "255"
        g_color = ET.SubElement(action, "G").text = "0"
        b_color = ET.SubElement(action, "B").text = "0" 
```

Creating a method for the second Smart View


    def third_rule(smartview, rules, building_storey): 

        for b in building_storey_list:
            rule = ET.SubElement(rules, "RULE")
    
            ifctype = ET.SubElement(rule, "IFCTYPE").text = 'Building Story'
            
            property = ET.SubElement(rule, "PROPERTY")
            property_name = ET.SubElement(property, "NAME").text= "Name"
            propertyset_name = ET.SubElement(property, "PROPERTYSETNAME").text = "Summary"
            property_type = ET.SubElement(property, "TYPE").text = "Summary"
            property_value_type = ET.SubElement(property, "VALUETYPE").text = "StringValue"
            property_unit = ET.SubElement(property, "UNIT").text = "None"
            
            
            condition = ET.SubElement(rule, "CONDITION")
            condition_type = ET.SubElement(condition, "TYPE").text = "StartsWith"
            condition_value = ET.SubElement(condition, "VALUE").text =  b #uilding_storey
            
            action = ET.SubElement(rule, "ACTION")
            action_type = ET.SubElement(action, "TYPE").text = "AddSetTransparent"
        
            r_color = ET.SubElement(action, "R").text = "255"
            g_color = ET.SubElement(action, "G").text = "255"
            b_color = ET.SubElement(action, "B").text = "255"    
            
The third rule method which visualizes all the building stories

![bcsv_screenshots_edit_smart_view](/images/bcsv_screenshot_smart_view_edit.png)
The script creates a XML which BimCollabzoom can read. 

```
def create_xml_declaration(file_path_xml):
    
    xml_declaration =   """<?xml version="1.0"?>
    <bimcollabsmartviewfile>
    <version>5</version>
    <applicationversion>Win - Version: 3.1 (build 3.1.13.217)</applicationversion>
</bimcollabsmartviewfile>
    """
    
    with open(file_path_xml, 'r') as xml_file:
        xml_data = xml_file.readlines()
        
        
    xml_data[0] = xml_declaration + '\n'
    
    with open(file_path_xml, 'w') as xml_file:
        xml_file.writelines(xml_data)    
```

Writing the XML file header. Please manually check if the XML declaration includes the corresponding bimcollabzoom version on your system.

```
def get_building_storey_and_spaces_zone():    
    
    zone_list = []
    
    for building_storey in building_stories:
        for i in (building_storey.IsDecomposedBy):
            for j in (i.RelatedObjects):
                for k in (j.HasAssignments):
                    if k.is_a('IfcRelAssignsToGroup'):
                        zone_list.append([building_storey.Name, j.Name, k.RelatingGroup.Name])
                
    return zone_list  

```

Getting a list of the Building Storey, Name and Relating Zone


```
def organize_zones_per_building_storey():

    zone_list = []
    
    #create building storey and zone list
    for i in get_building_storey_and_spaces_zone():
        for j in building_storey_list:
            if i[0].startswith(j): 
                (zone_list.append([j, i[2]]))
                
       
    #uniqify the list 
    unique_data = [list(x) for x in set(tuple(x) for x in zone_list)]   
    unique_zone_list = []
    
    for i in sorted(unique_data):
        unique_zone_list.append((i))
        
        
    #sort the zone list by building store
    zone_dict = defaultdict(list)
    
    for k, v in unique_zone_list:
        zone_dict[k].append(v)
        
    return zone_dict
```

Creating a zone dictionary which will be used to loop over in the ```create_xml``` method



```
for k, v in organize_zones_per_building_storey().items():
    create_xml(file_name='zone_filter_' + k + '.bcsv', building_storey=k, zone=v)  
```

Using the created dictionary to loop of the ```create_xml``` method.

Importing the XML files which BimCollabZoom calls 'bscv' files gives the following results

![bcsv_smart_view_sets](/images/bcsv_smart_view_sets.png)

The Smart View Sets (XML files) imported  in BimCollabZoom


![bcsv_smart_views](/images/bcsv_smart_views.PNG)

The Smart Views (XML file) imported in BimCollabZoom, what have happened is a filter for the IfcZone and colored coded it for each building storey.


![bcsv_smart_views](/images/bcsv_smart_views.PNG)

The result through a filtered Smart View.


![bcsv_screenshot](/images/bcsv_screenshot.png)

Filtering all the building numbers and bundling of spaces is possible without any expensive IFC software. Just with basic Python knowledge and the help of BimCollabZoom! 


The total script can be found [here](https://github.com/C-Claus/Miscellaneous/blob/master/create_space_filter.py)

-------

# Adding-properties-to-an-IFC-file
[< Back to top](##several-tooling-and-scripts-i-have-created)

```
import ifcopenshell 

file_name = 'C:\\Users\\CClaus\\Desktop\\Flat 11\\ruimtemodel_flat_11_chb.ifc'
ifcfile = ifcopenshell.open(file_name)

spaces = ifcfile.by_type('IfcSpace')
building_stories = ifcfile.by_type('IfcBuildingStorey')
owner_history = ifcfile.by_type('IfcOwnerHistory')[0]
chb_url = 'https://coenhagedoornbouw.github.io/_pages/bouwnummer_'

for building_storey in building_stories:
    for ifcrelaggregates in building_storey.IsDecomposedBy:
        for spaces in ifcrelaggregates.RelatedObjects:
            if len(spaces.Name.split('.'))  == 3:
                    nummer = (spaces.Name.split('.')[1])
            else:
                nummer = spaces.Name
                
            for ifc_relassigns_to_group in spaces.HasAssignments:
                if ifc_relassigns_to_group.is_a('IfcRelAssignsToGroup'):
                    if ifc_relassigns_to_group.RelatingGroup.Name == nummer:

                        p = ifcfile.createIfcPropertySinglevalue("URL", "URL", ifcfile.create_entity("IfcText", chb_url + ifc_relassigns_to_group.RelatingGroup.Name))  
                        property_set = ifcfile.createIfcPropertySet(spaces.GlobalId, owner_history, "Coen Hagedoorn Bouw", None, [p])             
                        ifcfile.createIfcRelDefinesByProperties(spaces.GlobalId, owner_history, 'Name', 'Description', [spaces], property_set)
                        
                        break
             
ifcfile.write(str(file_name.replace('.ifc', '_bouwnummer.ifc')))   
print (file_name.replace('.ifc', '_bouwnummer.ifc'))

```
