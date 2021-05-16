# Monoply

## Created by Catherine(Zhilin) Zhou and [Meredith(Xinru) Cheng](https://www.linkedin.com/in/meredith-cheng/)

In this project, we designed a board game called Monoply for multiple players with an AI computer player.

## Overview
The overall structure of this program splits into three sections: Model, View, and Control. We decide to implement the Bittersweet `MVC` design pattern to organize our classes and their relationships to make the program with higher cohesion inside the classes and lower coupling between classes.
### Model
- The model section of the program is centered at `Board`, which contains all the information on the game board and implements most of the actions during the game. It contains vectors of `shared_ptr` to the `Tile` objects, `Edge` objects, `Vertex` objects, and `Builder` objects. It has a composition relationship with all these classes. To minimize coupling and maximize cohesion, `Board` is the only class that holds the pointer to these objects, while other classes hold the index of these objects inside the vector. 
- `Builder` class holds all the information of a player, including the number of resources, the building points, and the vector of `Residence`(s). It has a composition relationship with the `Residence` class. 
- `Tile` class holds all the information of a tile on the game board, such as its number, value, the resource(s) it represents, and the vertices and edges it associates with. As mentioned above, the vector of `Vertex` and `Edge` contains the indicies instead of the object/pointer itself, to enhance programming design. 
- `Edge` and `Vertex` hold information of the edges and verticies of each tile.
- `Residence` class obtains information of each type of residence the player can build. The `UpdateResidence()` and `ImproveType()` function updates the Residence type and the building point of the current object.
### View
We use an abstract `Display` class with the subclass `TextDisplay` to implement features of the View section. `Display` contains `printBoard()`, `printResidences(int builder)`, and `printBuilders()`. When `Board` needs to print these information, it calls the corresponding function in the `Display` class to output the messages. We use the `Template Method` here, and we will discuss the details of this implementation in the `Design` section and `Resilience to Change` section later.

The `textDisplay` class holds the layout of the game board. We use the `rectangular_layout` text file as the template and add specific information to the template when the game starts. The `Board` gives the `Information` object it receives to initialize `Display`, and `Display` uses the informaiton stored to fill in the board. 

We use the `Observer` design pattern to implement the feautres in `Display`. Everytime `Board` makes a change to the state of `Builders`, `Vertices`, or `Edges`, it will notify `Display` by calling the corresponding functions. `Display` will then change the information it stores and the corresponding coordinates on the board. The `Board` can also potentially choose to attach or detach to different display layout.
### Control
The `Control` section consists of two parts: the `main` function and the `Controller` class. 
- `Main` contains a `Controller` object. It takes in input from the players and calls the corresponding `Controller` functions to implement the actions. It uses several exception structs to implement exception safe, making sure that the program will not crash or goes into error even if the user does not follow a legal sequence of actions.
- `Controller` goes between the `main` function and the `Board` class. It contains a pointer to the current `Board` and calls the corresponding `Board` functions when the `main` function calls its functions.
- We made an `Information` class to contain all the information the board needed at the beginning of the game. The `Information` class has a composition relationship with the `BuilderData` class, which contains the information of each builder from the loaded data. It takes in a `string` (from the given file) and translates them into the way that `Board` could read later.  
- We use `Template Method` to implement the features that the player gets to choose what type of board they want during run-time. We have an `Level` class and three subclasses, `RandomLevel`, `PreviousLevel`, and `CustomizedLevel`. `PreviousLevel` and `CustomizedLevel` both reads the file given by the player stores the information in the `Information` object. `RandomLevel` uses a `default_random_engine` to generate random sequences of the tiles and their values based on the setted distribution. The `Level` class uses the function `getBoard()` to return a `shared_ptr` to the `Board` object it constructs using the completed `Information` object. 

## Design
- At the start of the program, the user need to choose whether they want a random board or a customized board layout by giving the command-line instructions. We implement this feature by using the `Template method` on `Level`.


