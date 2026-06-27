# MERN SELF STUDY

## Question:

what is cascading style sheets (CSS), what it is used for, its basic syntax and properties... 

## Answer

CSS is a style sheet language used to describe the presentation and layout of a webpage written in HTML. CSS uses a ruleset structure to target HTML elements and apply styles to them.

**Purpose**

    • Styling: Controls colors, fonts, backgrounds, and text formatting.

    • Layout: Manages positioning, spacing, alignment, and responsive design for mobile screens.

**Basic Syntax**

    Core Properties

    • color: Changes text color (e.g., color: blue;).

    • background-color: Fills element background (e.g., background-color: #fff;).

    • font-size: Adjusts text size (e.g., font-size: 16px;).

    • margin: Creates space outside the element border.

    • padding: Creates space inside the element border.

    • color: Defines the text color of an element.

    • font-family: Specifies the font family to be used for text.

    • font-size: Sets the size of the text.

    • background-color: Defines the background color of an element.

    • margin: Controls the margin surrounding an element.

    • padding: Specifies the padding within an element.

    • border: Defines the border of an element.

    • width: Sets the width of an element.

    • height: Sets the height of an element.

    • display: Specifies how an element is displayed (e.g., block, inline).

*Example*:

`h1`   
{
    color: blue;  
    font-size: 24px;  
    background-color: #f0f0f0;  
    margin: 10px;  
    padding: 20px;  
    border: 1px solid black;  
    width: 300px;  
    height: 150px;  
    display: block;  
}  

**Core Ruleset Structure**

    • Selector: Points to the HTML element you want to style.

    • Declaration Block: Contains one or more declarations separated by semicolons, wrapped in curly braces {}.

    • Property: The style attribute you want to change.

    • Value: The specific setting you want to apply to the property.

*Example*  

`selector` {
  property: value;
}  

Breakdown of the Example:  

`selector`: This could be any valid CSS selector, such as h1 (targets all \<h1> elements) or .my-class (targets elements with the class my-class).

`property`: This specifies the visual aspect you want to style, such as color, font-family, or margin.

`value`: This defines the specific value for the chosen property. Values can be keywords (e.g., red), numerical values (e.g., 10px), percentages (e.g., 50%), or predefined constants.


**Selector Types**

    • Element Selector: Targets elements directly by their HTML tag name.

    • Class Selector: Targets elements with a specific class attribute. Uses a dot . prefix.

    • ID Selector: Targets a single element with a unique id attribute. Uses a hash # prefix.

*Examples*

/* Element Selector */
p {
  text-align: center;
}

/* Class Selector */
.button-blue {
  background-color: blue;
}

/* ID Selector */  
#main-header {
  font-size: 24px;
}  

**Multi-Property**
 
 *Example*

.card-profile {  
  background-color: #f0f0f0;  
  border: 1px solid #ccc;  
  padding: 20px;  
  margin-top: 15px;  
}

**Conclusion**

CSS is a fundamental building block of web development, providing developers with control over the visual presentation of web pages.  

Separation of content from presentation allows for the creation of visually appealing and consistent web designs. A solid understanding of CSS syntax and properties is essential for anyone involved in web development.