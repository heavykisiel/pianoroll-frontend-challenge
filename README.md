

## Demo
https://github.com/heavykisiel/pianoroll-frontend-challenge/assets/61516447/e814bef0-0b57-44ba-902d-b9a5ea8fc9dc

## Overview
This repository contains my submission for the coding challenge. The challenge required building an application that meets the following requirements:

## Requirements Part1

1. **Requirement 1:** Implement Grid Layout: Display the Piano Rolls in a responsive grid layout on the main page.

      Displaying following items in grid is not very difficult task, but keeping it responsive is different. I used media querries to change number of displayed columns.

2. **Requirement 2:** Interactive Selection: Make each Piano Roll in the grid clickable. Upon clicking an item, it should become the main element on the page, enlarging for better visibility.

      Each item of grid is clickable and upon clicking, elements are divided on two groups: activeList(maybe in future there will be need for 2 active items) and inactiveList. If its done then specific classes are attacked to elements and changing the way how website is displayed. I had to change display from grid to flex because I thought that it will be more difficult to implement scrollbar in when items are displayed in grid.

3. **Requirement 3:**  List Display: Simultaneously, display the rest of the Piano Rolls in a vertical list on the right side of the page, similar to how videos are listed on YouTube’s watch page. Ensure that the list is scrollable if there are more items than can fit on the screen.

      To create scrollbar I had to set the height of the item container equal to remaining space left using vh (view height). Then I added scroll to that. Additionally I added smooth scroll property for better user experience. I dont know if its good approach but when we change active element through sidebar it ensures that current active element going back to place where it was previously. I ensure that elements are not repeating. 
      
5. **Requirement 4:**  Styling: Apply styling to make the grid and its items visually appealing. Ensure that Piano Rolls are identifiable and have a consistent size across different screen resolutions.

      In the beggining I wanted to add colors for background and cards according to rule 90 : 30 : 10, but when I looked the results I thought it would be best to stay background as it is, to make website more vivid. Instead I set background color for cards to light spin-off of top bar color. I set margins based on viewport units to prevent website appearance looking unatractive on smaller resolutions. I added some animations that trigger once elements are loaded.
      
## Requirements Part2

1. **Requirement 1:** Start Selection: Initiate a selection when the user clicks on a point.

     Event that starting selection is mousedown that triggers only when activeItem's index equals selected item value. Then it gather the value of X of selected element.

2. **Requirement 2:** Show Visual Feedback: Provide real-time visual feedback as the user drags to extend the selection.

     To face that problem I used computed property to give user real-time feedback about the area that he selected. I had a problem that connected to event.target. It wasnt always class piano-roll-svg with active index that I wanted and it was glitchy cuz sometimes mouse took values from rectangles, lines and painted area inside of this container so I had to manage this cases too by accessing a parent container's properties.

3. **Requirement 3:**  Capture Selection Data: Once a selection is made, capture the data regarding the start and end points of the selection. It's enough to console.log the selection. Bonus points if you can get the number of notes within the selection.

     When selection is ended by mouseup or mouseleave then all rectangles with class note-rectangle are gathered and compared if they begin at the selected area or not. If they pass they are passed to array with notes. 
        
4. **Requirement 4:**  Visual Representation of Selection: Ensure that the selected range is clearly represented on the piano roll. You could change the color, add a border, or use any other visual cue to show the selection.

    After selection I added two lines that show where are start and end selection points are. You can enable or disable it with the button above. Points' value and notes.length numbers are displayed on the bottom of card.
   
## Room for improvement

This project has room for improvement and could benefit from the following changes and additional features:
  1. Divide code into smaller components.
  2. Make data on grid view appear with infinite scrolling feature (Only 32 from 2000 elements are displayed)
  3. Add more animations and some hover effects to the cards
  4. Improvements with CSS values.

Feel free to reach out to me with any questions or feedback at [nikolas99@wp.pl].

# PianoRoll Frontend Task

## Introduction

You will be building on top of a demo code that generates a widget for browsing Piano Rolls - the core element of [Piano Roll](https://pianoroll.io).
The goal is to enhance the user interface and add an interactive component.

This is what the starting point looks like:

<img width="666" alt="image" src="https://github.com/Nospoko/pianoroll-frontend-challenge/assets/8056825/daf35d32-f4e5-4a00-bbe8-78ecec2f2011">

## Task description

### Before you start

We recommend you start by understanding how the piano roll is structured and displayed, and what is the underlying data structure. If at any point you have any questions, you can join our [discord channel](https://discord.gg/2RadyyxADa) where we'll be providing active support.

Right now this is a Vanilla JS project. You can work on it as such, or you can use any framework you want - just make sure you document the process of running the application with all the requirements.

You can change, refactor or rewrite the existing code as much as you want.

### Part 1: Display

We want to implement an interface following the design similar to YouTube. There are two states we need to implement:

#### Grid layout

```
+--------------+   +--------------+   +--------------+
|              |   |              |   |              |
|  Piano Roll  |   |  Piano Roll  |   |  Piano Roll  |
|              |   |              |   |              |
+--------------+   +--------------+   +--------------+
+--------------+   +--------------+   +--------------+
|              |   |              |   |              |
|  Piano Roll  |   |  Piano Roll  |   |  Piano Roll  |
|              |   |              |   |              |
+--------------+   +--------------+   +--------------+
+--------------+   +--------------+   +--------------+
|              |   |              |   |              |
|  Piano Roll  |   |  Piano Roll  |   |  Piano Roll  |
|              |   |              |   |              |
+--------------+   +--------------+   +--------------+
                          ...
```

#### Main View

```
+-----------------------------------+  +--------------+
|                                   |  |              |
|                                   |  |  Piano Roll  |
|                                   |  |              |
|        Main Piano Roll            |  +--------------+
|                                   |  +--------------+
|                                   |  |              |
|                                   |  |  Piano Roll  |
+-----------------------------------+  |              |
                                       +--------------+
                                             ...
```

Here are development tasks to build this interface:

1. **Implement Grid Layout**: Display the Piano Rolls in a responsive grid layout on the main page.
2. **Implement Interactive Selection**: Make each Piano Roll in the grid clickable. Upon clicking an item, it should become the main element on the page, enlarging for better visibility.
3. **List Display**: Simultaneously, display the rest of the Piano Rolls in a vertical list on the right side of the page, similar to how videos are listed on YouTube’s watch page. Ensure that the list is scrollable if there are more items than can fit on the screen.
4. **Styling**: Apply styling to make the grid and its items visually appealing. Ensure that Piano Rolls are identifiable and have a consistent size across different screen resolutions.

### Part 2: Interactive Selection Tool

We want to give the user a possibility to select a fragment of the visible Piano Roll. This is a similar functionality to what you can see on [this prodigy demo](https://demo.prodi.gy/?=null&view_id=audio_manual). Here's our reference layout:

```
+-----------+-------------+---------+  +--------------+
|           |             |         |  |              |
|           |             |         |  |  Piano Roll  |
|           |             |         |  |              |
|           |             |         |  +--------------+
|           |             |         |  +--------------+
|           |             |         |  |              |
|           x1            x2        |  |  Piano Roll  |
+-----------+-------------+---------+  |              |
                                       +--------------+
                                             ...
```

Your task is to implement a tool or an overlay on the piano roll that allows users to click and drag to select a range. The tool should be able to:

1. **Start Selection**: Initiate a selection when the user clicks on a point.
2. **Show Visual Feedback**: Provide real-time visual feedback as the user drags to extend the selection.
3. **Capture Selection Data**: Once a selection is made, capture the data regarding the start and end points of the selection. It's enough to `console.log` the selection. Bonus points if you can get the number of notes within the selection.
4. **Visual Representation of Selection**: Ensure that the selected range is clearly represented on the piano roll. You could change the color, add a border, or use any other visual cue to show the selection.

## Submission

1. **Demonstration**: Fork this repository and work on your code there. When you're finished, update the readme with a screencast recording showing your version of the application.
2. **Submit Your Work**: Send us the link to your repository, add a link to your linkedin profile in the message. Submission email address: frontend@pianoroll.io

### Evaluation Criteria

1. **Functionality**: The selection mechanism should work as described, allowing users to select a range on the piano roll.
2. **User Experience**: The tool should provide a good user experience, with intuitive controls and clear visual feedback.
3. **Code Quality**: The code should be clean, easy to read, well-organized, and properly documented.



