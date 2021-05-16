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
 
   `Level` class contains a `shared_ptr` to an `Information` object. It uses the `getBoard()` function to generate a `shared_ptr` of `Board` by passing `info` to the `Board`'s constructor. `Level` has three subclasses: `RandomLevel`, `PreviousLevel`, and `CustomizedLevel`. 
        
  When the user gives the command-line instructions on what kind of board to create, `main` will pass these instructions to `Controller` by giving its constructor the seed and input files. The `Controller` will decide which \verb|Level| object to create at runtime. Since for whatever type of `Board` to create, they all need an `Information` object and pass it to the constructor. Thus, we decide to use the `Template Method`, such that he three `Level` subclasses have different implementation on `updateInfo()`, ad the `Level` class has a public `getBoard()` function. Then at run-time, `Controller` will get the corresponding board by calling `getBoard()`. 
- At the beginning of the turn, the player chooses to roll a loaded dice or a fair dice. We implement this feature by using the `Factory Method`, that we have an abstract `Dice` class and two subclasses (`LoadedDice` and `FairDice`).
    
    We will determine which object to create at run-time. The `Controller` class passes the `seed` parameter to `FairDice` and the player-chosen number to `LoadedDice`. Both classes will then return the rolled number by the overriden `rollDice()` function, either using a `default_random_engine` to shuffle the dice, or directly return the chosen number. To make sure that the rolled number is random while using the `FairDice`, we shuffle the list of numbers (1 to 6) two times, and take the first element on the shuffled list as the index of the number rolled in the shuffled list. We then add the two results we have to return.
    
- During the game, each builder will make changes to their state by rolling a dice (gaining resources), building a road, or building a residence, etc. We want to udpate these changes immediately to the `Board` as well as the `Display`, since the player can choose to print the board or player status right after they nake these changes. Therefore, we use the `Observer` design pattern here to implement this feature. Everytime the builder makes a change, the `Controller` calls the corresponding function in `Board` to conduct this action. `Board` will then notify the `Display` object it attached to at the beginning of the game to make the corresponding updates. Subsequently, when the player wants to print the board, `Display` can immediately print the up-to-date information.

- During the game, each builder can choose to build a basement or a road, which definitely changes their state on the resources they hold, as well as the state of the board. We adopt the `Observer` design pattern here to implement this feature. We contain mappings from verticies to their adjacent vertices and edges in `Vertex` and `Edge`. Everytime the builder build a road or makes a change to their residences, `Board` will notify the adjacent verticies and edges of the corresponding vertex or edge to make some changes to their state. We decide to add the builder's name to these adjacent verticies or edges to show that such builder has built a basement or road. Doing so helps us keep track on the current state of each vertex and edge, so that when the builder want to build a road or another basement later, we can check their validity using the most up-to-date information.

- We noticed that this program requires a lot of interaction with the user, by taking in input from the user, conduct actions, and output the corresponding messages to the user. The entire process matches the `MVC` design pattern, and thus, we decide to adopt this design pattern to implement the user-interaction features of this game. 
    
    We separate our classes into three parts: `Model`, `View`, and `Control`. The `Control` section includes classes that takes in user input, translates it into the format that is understandable by other functions, and calls the corresponding functions in the `Model` section. It acts like the deliver between the user and the program itself. The `Model` section includes classes that actually conduct the instructions and calculates the correct output. After the calculations are done, it delivers the raw result to the `View` by updating the output information stored in `View`. The `View` section then prints the message or shows the display to the user, letting them know how their instructions are fulfilled. We have the follwoing example to illustrate how we use this design pattern to enhance our program:

    One of the features of this game is Geese. When the dice is rolled to a 7, `Control` calls the `Geese()` function in `Model`. `Model` first finds a list of builders that have at least 10 resources on hand and give this list to `View` to let these players know how many of their resources have been lost to the Geese. `View` then asks the current builder to choose a new position to place the Geese. `Control` takes the chosen vertex and pass it to `Model` by calling the `UpdateGeese(new location)` function. `Model` searches for the builders that have built residences on such tile and delivers the list of such builders to `View`. If there are no builders to steal, then `View` will just output the message, and `Control` will move on to the next instruction. If the list is not empty, then `View` asks the player to choose one builder from the printed list. `Control` gets the name from the player and passes it to `Model`. `Model` generate a randon resource from the chosen builder and gives the resource to the current builder. It updates `View` with this change on state, and `View` then prints the message to notify the builders. 
    
## Resilience to Change    
- The player might want to switch to a different display, such as a graphic display, in the middle of the game (at run-time). Considering this potential change, we use the `Template Method` on the `Display` class.
  We have a `Display` class, which contains the current up-to-date information about the builders, vertices, and edges, including mapping from builder to their number of resources and residences, mapping from verticies to their position in the board layout, etc. The information can be accesses by all its subclasses, so that they could construct their layout and update the information when get notified by the `Board`. 
    `Display` class contains public functions to print error messages or output results from the `Board`. It also contains pure virtual functions for updates and printing. We choose to leave them as pure virtual functions, since the different subclasses will have different implementation on how to update the information as well as printing the layouts.
    
    Inheriting `Display`, the `TextDisplay` class obtains a vector of strings (we use vector of chars here, so that it will be easy for us to access and update the layout). It implements the virtual methods from `Display`. Since the `Board` only contains a `Display` object and only decide which specific display type it is when constructing this object at the beginning, both the non-virtual methods and the virtual methods will work at run-time, and the virtual methods will have different display output when using differerent display type. 

    We originally plan to add another display type such as changing the board layout, but due to the time constraint, we didn't implement such feature. However, we still keep the `Template Method` design pattern, just so we can always implement such features with the smallest amount of change to our program -- we only need to add another subclass to the `Display` class and change the implementation of the virtual methods in the new subclass.
    
- The players might want to change the settings of the game when restarting the game at the end. For exmaple, they originally set the game to a previous game status, and after one player wins, they decide to use another board layout or game status instead of the original one. To implement this potential change, we use the `Template Method` on setting the board. 
    
    At the beginning of the game, when the player gives the command-line instructions, we store the information we get from the file in the `Information` and `BuilderData` class. We have a `Level` class and three subclasses to separate the cases when the command-line is 'random', 'board' or 'load'. `Level` class has a public `getBoard()` method. Calling which will return a `shard_ptr` to the `Board` with the stored `Information`. It contains a private virtual `updateInfo()` method, and the three subclasses all have different implementation on this method. When `Controller` calls `getBoard()`, the `Board` it returns depends on the type of `Level` object `Controller` creates based on the user input. 

    `Controller` holds the `Level` object it originally creates, so when the game finishes and restarts, it can use the public `restart()` method in `Level`, which returns a new `Board` pointer with the same `Information` as the beginning. Thus, if we want to implement this change, we only need to change the `restart()` method in `Level` and the `main` function at the end, which will ask the players to input a new command-line instruction. `Controller` will decide the new type of `Level` object it needs to create and pass the corresponding information to this new object. Then, calling the new `getBoard()`, it shall return the udpated `Board` with the new settings the players just choose.    

- We add a new feature to the game called `Bank`. The detail of this feature will be explianed in the extra-credit section. We did not change our program a lot when adding this new feature because we adopted the `Observer` design pattern in `Model`. 
    
    When the builder want to mortage his/her residence to the bank, `Board` will notify the current `Builder`, and `Builder` will detach the corresponding `Residence` at the given vertex. The detach function will first delete this `Residence` from the vector of residences stored in the `Builder`. It will then notify the vertex on which the `Residence` locates and the adjacent verticies and edges to make changes to their states. 

    Adopting the `Observer` design pattern divides the work of implementing each instruction into different parts, which will be done by different classes. This maximizes the cohesion within a class and minimizes the coupling of different classes.
    
- We add another feature to the game such that the player can choose the number of builders in the game by inputing `-players x`, where $x$ is the number of players. We make minimal changes to our program since we implement the `MVC` design pattern. Since we separate our classes and functions into three sections, making change to the number of players barely changes the `Model` section. We only make minimal change to `main`, `controller`, and `Display` to add player size as a private member and change the vectors of players into the correct size. 
    
    The reason why we almost do not need to change the `Model` section is that everytime `Controller` calls a `Board` function, it passes the index of the current player to `Board`, such that the `Board` always uses the correct index of player to conduct the following actions, and will not get an invalid player index. `Controller` keeps track of the number of players and the index of the current player to make sure that the index number it gives to `Board` is valid. The only thing we need to change in `Model` is to add player size as a private member, so that when `next()` is called, it can change `curTurn` to the correct player. We did not give much changes to `Display` either. We only add player size as a private member and change the vectors' size to match the player's size.
    
