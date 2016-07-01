# Simple-Xml
A small and very simple to use C library for reading XML files. Can be used simply by including a single header file.

# Usage
This library is still work in progress and I'd currently not really recommend using it as it lacks some features and has barely been tested. I am likely to change the signatures of some functions in future versions once I've given more thought to the usage of the library so don't expect forward compatibility. It should however function fine for simple applications if you don't mind it's use of the standard library. (Just stdio.h, stdlib.h and string.h)

To add the library to a project simply include the header file: "xmlReader.h". You can then use the following functions:

Note:
Currently comments and CDATA tags are not supported by the library. I will implement these at some later point in time. The library refers to a single tag/element as a node. Properties of that element/tag are refered to as attributes. e.g. <node attribute="value"></node>

**void xmlDocumentLoad(XmlDocument* document, char* filePath)**
Loads an document at the filepath passed. The document is loaded into the passed in XmlDocument structure.
  
**XmlNode* xmlNodeFindChildWithName(XmlNode* n, char* name)**
Finds a child node within the passed in node that has a matching name.
Returns a pointer to the node if one was found. Returns NULL or 0 if no matching node was found.

**int xmlNodeGetAllChildrenWithName(XmlNode* node, char* name, XmlNode\*\* destinationArray, int max)**
Gets all the child nodes within a node that match the passed in name. The function outputs pointers to all matches into the passed in "destinationArray" array. You must pass in the size of the array in the "max" parameter. The function returns a count of the number of matching elements that were found.

**XmlNode* xmlDocumentQuery(XmlDocument* d, char* query)**
This function returns the node within the the document that matches the passed in query. If a node that matches the query is not found 0 or NULL is returned. Queries currently only support a very simple syntax that basically represents a path to the desired node. e.g. "food.fruit.apple" would return the apple element from the following example xml:
```
<food>
  <veg>
    <potato></potato>
    <leek></leek>
  <fruit>
    <apple><apple>
    <banana></banana>
  </fruit>
</food>
```

**XmlNode* xmlNodeQuery(XmlNode* node, char* query, XmlDocument* d)**
This function works exactly the same as the xmlDocumentQuery function except the query is performed relative to the passed in Node.

**char* xmlNodeGetAttribute(XmlNode* n, char* name)**
Finds the value of the attribute that has a matching name within the passed in node. If the attribute does not exist within the node 0 or NULL is returned.

**void xmlDocumentFree(XmlDocument* d)**
Frees all memory used by the loaded XML document. In doing so all nodes are also freed so all reading of the document must be finished before this function is called.

**char* xmlNodeGetText(XmlNode* n)**
Returns the text that was within the passed in node. Note: If you wish to retain this string after xmlDocumentFree has been called you must make your own copy of the string. For example by doing the following:
```
  char* text = xmlNodeGetText(node);
  char* textCopy = (char*)malloc(strlen(text) + 1);
  strcpy(textCopy, text);
```

#Example
The following is a very simple example program that reads a very simple XML document:

Example test.xml:
```
<outer width="200" height="439">
    <inner id="1">
        <selfclose attrib="a" attrib2='b' />
        <value>This is a value</value>
        <value>This is another value</value>
    </inner>
    <inner id="2">
        <value>This is a third value</value>
        <value>This is a fourth value</value>
    </inner>
</outer>
```

Example main.c:
```
#include "xmlReader.h"
#include "stdio.h"

int main(int argc, char** argv)
{
    XmlDocument document;
    // Load the document
    xmlDocumentLoad(&document, "test.xml");
    
    // Find the first "inner" element.
    XmlNode* innerElement = xmlDocumentQuery(&document, "outer.inner");
    // Output the value of the id attribute of element "inner"
    printf("Found node \"%s\" with id \"%s\"\n", innerElement->name, xmlNodeGetAttribute(innerElement, "id"));
    
    // Find all elements that are called "value" within the previously found "inner" element and 
    // output the text within them to the console.
    XmlNode* nodeList[32];
    int count = xmlNodeGetAllChildrenWithName(innerElement, "value", nodeList, 32);
    int i;
    printf("Found %d value nodes within inner\n", count);
    for(i = 0; i < count; ++i)
        printf("Found a value: %s\n", xmlNodeGetText(nodeList[i]));
    
    xmlDocumentFree(&document);
    
    return 0;
}
```
