# Chess-Titans

[Live Site](https://punith-kk.github.io/Chess-Titans/forest-landing-page/dist/index.html)

[UCI using StockFish-13](https://punith-kk.github.io/Chess-Titans/chess-titans-uci/)

[My Project on Devpost](https://devpost.com/software/chess-titans)

# UCI-Protocol Implementation
## Check chess-titans-uci directory for programs implementation.

A simple [UCI](https://en.wikipedia.org/wiki/Universal_Chess_Interface) (*Universal Chess Interface*) Client written in Java.

Tested with [Stockfish 13](https://stockfishchess.org/blog/2021/stockfish-13/).

Maven:

```xml
<dependency>
  <groupId>net.punith</groupId>
  <artifactId>chess-titans</artifactId>
  <version>1.0</version>
</dependency>
```
## UCI Stockfish Integration 🎲

In the latest update of Chess-Titans, we've introduced an exciting new feature - the UCI Stockfish Integration! With this addition, Chess-Titans now harnesses the powerful AI capabilities of the renowned Stockfish chess engine to offer an even more challenging and engaging gameplay experience.

**What's New:**

🔹 **Stockfish-powered AI Opponent:** The CPU opponent in the "VS CPU" mode has been upgraded to use Stockfish as its underlying engine. Stockfish is well-known for its exceptional strength and strategic prowess, making it a formidable adversary for players seeking a tough chess challenge.

🔹 **Adjustable Difficulty Levels:** With the UCI Stockfish Integration, players can now fine-tune the AI opponent's difficulty level to match their skill and expertise. Whether you're a casual player looking for a fun match or a seasoned chess enthusiast seeking a real test of your abilities, Chess-Titans has the perfect difficulty setting for you.

🔹 **Enhanced Game Logic:** The integration of Stockfish required us to rework and optimize various aspects of the game's logic. From evaluating positions to calculating possible moves, the chess engine has been fine-tuned to deliver an unparalleled playing experience.

🔹 **Seamless UCI Protocol Communication:** The interaction between Chess-Titans and Stockfish is carried out flawlessly using the UCI (Universal Chess Interface) protocol. This ensures smooth communication, enabling the game to leverage Stockfish's immense computational power while maintaining compatibility with the Arena GUI and other UCI-compatible interfaces.

**What's Next:**

We're thrilled with the inclusion of the UCI Stockfish Integration, but our journey doesn't end here. We're committed to continuously enhancing Chess-Titans and delivering the best possible chess experience to our players. In the upcoming updates, we plan to introduce additional features like online multiplayer functionality, new visual themes, and advanced analytics to further enrich the game.

Thank you for your continued support, and we hope you enjoy the enhanced chess gameplay experience with Chess-Titans! 🌟🏆🚀

# Documentation 

## Starting / Closing the client

By using the `startStockfish()` method, the client assumes that Stockfish is already installed on the system, and it's accesible in `$PATH` as `"stockfish"`.

```java
var uci = new UCI();
uci.startStockfish();
```        

Using `start(String cmd)` **chess-titans** can be tested with other chess engines:

```java
// chess-titans
var uci = new UCI();
uci.start("lc0");
```

By default each command you are sending to the engine, has a timeout of `60s` (during which the thread is blocked).

You can configure the global default timeout to a different value:

```java
var uci = new UCI(5000l); // default timeout 5 seconds
uci.startStockfish();
```

If the client commands exceed the timeout interval an unchecked `UCITimeoutException` is thrown. But more on that in the next sections.

To retrieve the `defaultTimeout` simply call: `var timeout = uci.getDefaultTimeout()`.

To close the client simply call: `uci.close();`.

## Commands

Most commands respond with an `UCIResponse<T>` object. This class wraps a possible exception, and the actual result value.

The most common idiom(s) is/are:

```java
var response = uci.<some_command>(...);
var result = response.getResultOrThrow();
// do something with the result
```

or

```java
var response = uci.<some_command>(...);
if (response.succes()) {
  var result = response.getResult();
  // do something with the result
}
```

An `UCIResponse<T>` can only throw a `UCIRuntimeException`. This class has the following sub-types:
- `UCIExecutionException` - when the `Thread` executing the command fails;
- `UCIInterruptedException`- when the `Thread` executing the command fails;
- `UCITimeoutException` - when `Thread` exeucuting the command timeouts;
- `UCIUncheckedIOException` - when the communication with the engine process fails;
- `UCIUnknownCommandException` - when the server doesn't understand the command the client has sent;
- `UCIParsingException` - when the engine sends output that the client doesn't understand;

All commands support custom timeout. Usually this is the last parametere of the command function:

```java
var response = uci.<some_command>(args, long timeout);
```

## Retrieving the engine information

The method for obtaining the engine information (the current engine name and supported engine options) is `UCIResponse<EngineInfo> = uci.getEngineInfo()`.

Example:

```java
var uci = new UCI(5000l); // default timeout 5 seconds
uci.startStockfish();
UCIResponse<EngineInfo> response = uci.getEngineInfo();
if (response.success()) {

    // Engine name
    EngineInfo engineInfo = response.getResult();
    System.out.println("Engine name:" + engineInfo.getName());
    
    // Supported engine options
    System.out.println("Supported engine options:");
    Map<String, EngineOption> engineOptions = engineInfo.getOptions();
    engineOptions.forEach((key, value) -> {
        System.out.println("\t" + key);
        System.out.println("\t\t" + value);
    });
}
uci.close();
```

Output:

```
Engine name:Stockfish 13
Supported engine options:
	Hash
		SpinEngineOption{name='Hash', defaultValue=16, min=1, max=33554432}
	Move Overhead
		SpinEngineOption{name='Move Overhead', defaultValue=10, min=0, max=5000}
	UCI_AnalyseMode
		CheckEngineOption{name='UCI_AnalyseMode', defaultValue=false}
	UCI_LimitStrength
		CheckEngineOption{name='UCI_LimitStrength', defaultValue=false}
	Threads
		SpinEngineOption{name='Threads', defaultValue=1, min=1, max=512}
	MultiPV
		SpinEngineOption{name='MultiPV', defaultValue=1, min=1, max=500}
/// and so on
```

## Setting an option

For changing an option of the engine you can use the following methods:
- `UCIResponse<List<String>> setOption(String optionName, String value, long timeout)`
- `UCIResponse<List<String>> setOption(String optionName, String value)`

For example, modifying the `MultiPV` option to `10` is as simple as:

```java
var uci = new UCI(5000l); // default timeout 5 seconds
uci.startStockfish();
uci.setOption("MultiPV", "10", 3000l).getResultOrThrow(); // custome timeout 3 seconds
uci.close();
```

## Getting the best move for a position

If you plan using the engine to analyse various positions that are not part of the same game, it's recommended to call the `uciNewGame()` first.

An UCI engine understands [FEN notations](https://en.wikipedia.org/wiki/Forsyth%E2%80%93Edwards_Notation), so the method to make the engine aware of the position he needs to analyse is: 
- `UCIResponse<List<String>> positionFen(String fen, long timeout)`
- `UCIResponse<List<String>> positionFen(String fen)`

After the position has been set on the board, to retrieve the best move:
- `UCIResponse<BestMove> bestMove(int depth, long timeout)` - to analyse for a given depth (e.g.: 18 moves deep);
- `UCIResponse<BestMove> bestMove(int depth)`;
- `UCIResponse<BestMove> bestMove(long moveTime, long timeout)` - to analyse for a fixed amount of time (e.g.: 10000l - 10 seconds);
- `UCIResponse<BestMove> bestMove(long moveTime)`;

Let's take the example the following position:

<img width="40%" alt="position01" src="https://github.com/Punith-KK/Chess-Titans/assets/118302022/e674e557-9578-4e17-865e-b606d7fb0c3b">

The corresponding FEN for the position is:

```
rnbqk3/pp6/5b2/2pp1p1p/1P3P1P/5N2/P1PPP1P1/RNBQKB2 b Qq - 0 14
```

The code that determines what the best move is:

```java
var uci = new UCI(5000l); // default timeout 5 seconds
uci.startStockfish();
uci.uciNewGame();

uci.positionFen("rnbqk3/pp6/5b2/2pp1p1p/1P3P1P/5N2/P1PPP1P1/RNBQKB2 b Qq - 0 14");

var result10depth = uci.bestMove(10).getResultOrThrow();
System.out.println("Best move after analysing 10 moves deep: " + result10depth);

var result10seconds = uci.bestMove(10_000l).getResultOrThrow();
System.out.println("Best move after analysing for 10 seconds: " + result10seconds);

uci.close();
```

## Analysing a position

Analysing the best N lines for a given FEN position is very similar to the code for finding what is best move. 

The methods for finding out what are the best lines are:
- `UCIResponse<Analysis> analysis(long moveTime, long timeout)` - to analyse for a given depth (e.g.: 18 moves deep);
- `UCIResponse<Analysis> analysis(long moveTime)`
- `UCIResponse<Analysis> analysis(int depth, long timeout)` - to analyse for a fixed amount of time (e.g.: 10000l - 10 seconds);
- `UCIResponse<Analysis> analysis(int depth)` 

> By default Stockfish analyses only one line, so if you want to analyse multiple lines in parallel, you need to set `MultiPV`: `uci.setOption("MultiPV", "10", 3000l)`

Let's take for example the following position:

<img width="40%" alt="position02" src="https://github.com/Punith-KK/Chess-Titans/assets/118302022/f1746f3e-e2c8-4bcd-aa57-5d2c7012c32d">

The corresponding FEN for the position is:


```
r1bqkb1r/2pp1ppp/p1n2n2/1p2p3/4P3/1B3N2/PPPP1PPP/RNBQK2R w KQkq - 2 6
```

And in order to get the 10 best continuations, the code is:

```java
var uci = new UCI();
uci.startStockfish();
uci.setOption("MultiPV", "10");

uci.uciNewGame();
uci.positionFen("r1bqkb1r/2pp1ppp/p1n2n2/1p2p3/4P3/1B3N2/PPPP1PPP/RNBQK2R w KQkq - 2 6");
UCIResponse<Analysis> response = uci.analysis(18);
var analysis = response.getResultOrThrow();

// Best move
System.out.println("Best move: " + analysis.getBestMove());
System.out.println("Is Draw: " + analysis.isDraw());
System.out.println("Is Mate: " + analysis.isMate());

// Possible best moves
var moves = analysis.getAllMoves();
moves.forEach((idx, move) -> {
    System.out.println("\t" + move);
});

uci.close();
```

The output:

```
Best move: 
	Move{lan='e1g1', strength=0.6, pv=1, depth=18, continuation=[c8b7, d2d3, f8c5, b1c3, ...]}
Best moves:
	Move{lan='e1g1', strength=0.6, pv=1, depth=18, continuation=[c8b7, d2d3, f8c5, b1c3, ...]}
	Move{lan='d2d4', strength=0.55, pv=2, depth=18, continuation=[d7d6, c2c3, f8e7, e1g1, ...]}
	Move{lan='a2a4', strength=0.52, pv=3, depth=18, continuation=[c8b7, d2d3, b5b4, b1d2, ...]}
	Move{lan='b1c3', strength=0.36, pv=4, depth=18, continuation=[f8e7, d2d3, d7d6, a2a4, ...]}
	Move{lan='d2d3', strength=0.26, pv=5, depth=18, continuation=[f8c5, b1c3, d7d6, c1g5, ...]}
	Move{lan='d1e2', strength=-0.02, pv=6, depth=18, continuation=[f8c5, a2a4, a8b8, a4b5, ...]}
	Move{lan='c2c3', strength=-0.50, pv=7, depth=18, continuation=[f6e4, e1g1, d7d5, f1e1, ...]}
	Move{lan='b3d5', strength=-0.57, pv=8, depth=18, continuation=[f6d5, e4d5, c6d4, f3d4, ...]}
	Move{lan='h2h3', strength=-0.63, pv=9, depth=18, continuation=[f6e4, e1g1, d7d5, b1c3, ...]}
	Move{lan='f3g5', strength=-0.7, pv=10, depth=18, continuation=[d7d5, d2d3, c6d4, e4d5, ...]}
```






[Youtube Video Link](https://youtu.be/Vx-SCf2P-ro)


![parallax](https://github.com/Punith-KK/Chess-Titans/assets/118302022/5a55f2f0-cefc-4138-9148-1bb86104afc2)
![effects](https://github.com/Punith-KK/Chess-Titans/assets/118302022/ab627c45-af42-461b-8680-e0b2867bb870)
![all](https://github.com/Punith-KK/Chess-Titans/assets/118302022/dca0f208-a6a7-4c3d-89a4-e510cd4151b1)
![chess](https://github.com/Punith-KK/Chess-Titans/assets/118302022/86051cf5-c5e6-4179-9c05-69472fc5200e)




![1](https://github.com/Punith-KK/Chess-Titans/assets/118302022/7cf21fa8-bb2d-471d-bb3b-60f67ff5513c)
![2](https://github.com/Punith-KK/Chess-Titans/assets/118302022/c96afd76-279d-4a13-b155-57432cf477f4)
![3](https://github.com/Punith-KK/Chess-Titans/assets/118302022/cfede01c-b807-4e63-9480-af25a1c8f99c)
![4](https://github.com/Punith-KK/Chess-Titans/assets/118302022/97b6c65c-8cfb-4680-bb44-106860d1fe4e)
![5](https://github.com/Punith-KK/Chess-Titans/assets/118302022/c3b3efd8-277a-4337-ac89-a280a6a71259)
![6](https://github.com/Punith-KK/Chess-Titans/assets/118302022/2a1052c1-bb25-49b6-9ae7-99624bd191e6)
![7](https://github.com/Punith-KK/Chess-Titans/assets/118302022/03013da4-af9d-410a-b32b-e2c70bacfac1)
![8](https://github.com/Punith-KK/Chess-Titans/assets/118302022/d67c031b-1549-4725-a4e8-be3ec68919c9)
![9](https://github.com/Punith-KK/Chess-Titans/assets/118302022/eeb20c3b-73d7-4de0-9dbe-13dbf4a70b46)
![10](https://github.com/Punith-KK/Chess-Titans/assets/118302022/2f2fc754-9343-44b7-9aa6-28e42d04a2cd)
![chess2](https://github.com/Punith-KK/Chess-Titans/assets/118302022/fb3212e1-dc79-4b5b-bef1-c4ea08529063)


# Chess-Titans: ♟️🏰 A 3D Chess Game with Parallax Effects 🌌🌟

## About the Project

Chess-Titans is a unique and visually captivating chess game that I developed for a hackathon project. The inspiration behind this project came from my passion for chess ♟️🤩 and my desire to create a modern and immersive chess experience for players of all skill levels. I wanted to combine my knowledge of web development (HTML, CSS, JS, SCSS) with 3D effects and parallax scrolling to make a visually stunning and interactive chess game.

## Problem Statement

The objective of the project was to create a fully functional and visually appealing chess game with the following features:

1. **Chess Gameplay**: Implement the core chess logic to allow players to move pieces, check for valid moves, and implement game-ending conditions (checkmate, stalemate).

2. **VS Human and VS CPU Modes**: Provide options for players to choose between playing against another human player or challenging an AI-based CPU opponent with varying difficulty levels.

3. **Login Page**: Design a secure login page where users can create accounts, log in, and save their gameplay progress.

4. **Official Website**: Develop an official website for the Chess-Titans game where users can learn about the game, access the login page, and download the game if needed.

5. **3D Chess Board Modal**: Create a visually stunning 3D chessboard modal with realistic animations and intuitive controls.

6. **Parallax Effects**: Implement parallax scrolling effects on the website to enhance user engagement and provide a captivating visual experience.

## My Contribution

### Chess Logic and Game Development

I started by building the core chess logic in JavaScript. This involved defining the rules for each chess piece, validating moves, checking for checkmate and stalemate conditions, and handling the game state. I also created the necessary HTML and CSS structure for the chessboard and pieces.

### VS Human and VS CPU Modes

For the VS Human mode, I implemented the functionality to enable two players to make moves alternately by using event listeners and handling user input.

For the VS CPU mode, I integrated a chess engine based on the minimax algorithm with alpha-beta pruning. I fine-tuned the algorithm's depth to control the difficulty level of the CPU opponent, allowing players to choose between easy, medium, and hard difficulty settings.

### Login Page and User Authentication

I designed a sleek and secure login page using HTML and CSS. To handle user authentication, I utilized JavaScript to manage user accounts, store data securely, and allow users to save their progress and access it later. I also implemented encryption techniques to ensure data privacy.

### Official Website

For the official website, I used HTML, CSS, and SCSS to create an attractive and user-friendly interface. The website showcased the game's features, displayed screenshots, and provided a download link to the game.

### 3D Chess Board Modal and Parallax Effects

The most exciting part of the project was creating the 3D chessboard modal with parallax effects. I utilized Three.js, a popular JavaScript library for 3D rendering, to build the 3D chessboard. I added smooth animations to chess piece movements and integrated parallax effects on the website to enhance the overall visual appeal.

## Challenges Faced

1. **Chess Logic Complexity**: Implementing the intricate rules of chess and ensuring all moves were correctly validated required careful planning and rigorous testing.

2. **AI Development**: Designing and fine-tuning the AI algorithm to provide a challenging but beatable CPU opponent was a challenging task. Balancing difficulty levels was crucial to offer an enjoyable experience for players of varying skill levels.

3. **3D Chessboard and Parallax Integration**: Learning and implementing Three.js to create a realistic 3D chessboard with smooth animations was challenging. Additionally, incorporating parallax effects on the website without negatively impacting performance required optimization.

4. **User Authentication and Data Security**: Ensuring user data privacy and implementing secure authentication mechanisms demanded meticulous attention to detail.

## What I Learned

Developing Chess-Titans was an incredible learning experience. I honed my skills in web development, gained proficiency in JavaScript, and deepened my understanding of CSS, SCSS, and 3D rendering with Three.js. I also gained valuable insights into AI algorithms and user authentication techniques.








## Inspiration 🎯

The inspiration for Chess-Titans came from my deep passion for chess ♟️ and my love for modern web technologies. I wanted to create a chess game that not only showcased the timeless elegance of the game but also pushed the boundaries of what could be achieved with web development. The idea of combining 3D effects, parallax scrolling, and a powerful AI opponent ignited my creativity and drove me to create Chess-Titans.

## What it does 🕹️

Chess-Titans is a feature-rich chess game that offers two gameplay modes: VS Human and VS CPU. In VS Human mode, players can compete against each other on a visually stunning 3D chessboard. Alternatively, in VS CPU mode, they can challenge an AI-based CPU opponent with adjustable difficulty levels.

The game also includes a secure login page that allows users to create accounts, log in, and save their gameplay progress. Additionally, an official website showcases the game's features, provides download options, and delivers a captivating experience through parallax effects.

## How we built it 🛠️

We built Chess-Titans using a combination of HTML, CSS, JavaScript, and SCSS for the frontend. For the 3D chessboard, we employed the powerful Three.js library to create a realistic and visually appealing game environment. The AI opponent was implemented using the minimax algorithm with alpha-beta pruning to provide challenging gameplay.

The login page and user authentication were designed to ensure data security and privacy. We utilized encryption techniques to safeguard user data and enable seamless account management.

## Challenges we ran into 🚧

Building Chess-Titans came with its share of challenges. Implementing the intricate chess logic to handle valid moves, checkmate conditions, and stalemates required careful planning and extensive testing.

Integrating the AI opponent with adjustable difficulty levels was a significant challenge. Fine-tuning the algorithm to strike the right balance between challenging and beatable AI opponents demanded iterative optimization.

Creating the 3D chessboard modal with smooth animations and parallax effects was both thrilling and technically demanding. Ensuring optimal performance while rendering the 3D environment and handling user interactions added an extra layer of complexity.

## Accomplishments that we're proud of 🏆

We are immensely proud of successfully developing Chess-Titans, a visually captivating and interactive chess game that brings the joy of chess to players worldwide. Our achievement lies not only in building the game but also in combining cutting-edge web technologies with a classic board game to create a unique and immersive experience.

The seamless integration of a secure login page and user authentication features ensures that players can enjoy the game while having their progress and achievements safely stored.

## What we learned 📚

Throughout the development of Chess-Titans, we deepened our knowledge of web development, JavaScript, CSS, and SCSS. The implementation of the AI opponent using the minimax algorithm enhanced our understanding of game theory and artificial intelligence.

Working with Three.js to build the 3D chessboard exposed us to the world of 3D rendering and animation in web development. We gained valuable insights into optimizing performance in WebGL applications.

Moreover, the project taught us the significance of meticulous planning, problem-solving, and the joy of creating a project that reflects our passion for both chess and web development.

## What's next for Chess-Titans 🚀

The journey for Chess-Titans doesn't end here. We have exciting plans for future enhancements, such as adding online multiplayer functionality to allow players from all over the world to compete against each other. Additionally, we aim to introduce new themes and customizations to elevate the visual experience.

We'll continuously improve the AI opponent's capabilities and explore machine learning techniques to make it even more challenging and dynamic. As the game gains popularity, we'll actively seek user feedback and implement requested features to provide the best possible gaming experience.

Chess-Titans will continue to evolve and remain a source of joy for chess enthusiasts and players everywhere. 🌟🌍🏆


## Conclusion

Chess-Titans turned out to be a successful and rewarding project. Combining my passion for chess with modern web technologies and visual effects allowed me to create a game that stood out in both functionality and aesthetics. The project taught me the importance of perseverance, problem-solving, and the joy of creating something innovative and visually appealing. I hope Chess-Titans brings joy to chess enthusiasts and players worldwide. 🌟🌍🎉

## Instructions to Run Chess-Titans 🚀

Follow these steps to run Chess-Titans on your local machine:

1. **Clone the Repository** 📋: Clone the Chess-Titans repository from GitHub using the following command:
   ```bash
   git clone https://github.com/Punith-KK/Chess-Titans.git ```

2. **Navigate to the Project Directory** 📂: Change your current working directory to the Chess-Titans project folder:

```bash
cd Chess-Titans/forest-landing-page 
```

3. **Run the Game** ▶️: Open the project folder and locate the index.html file. Simply double-click on the index.html file to open it in your default web browser.

### Start Playing ♟️🎮: The Chess-Titans game will load in your web browser, and you can start playing immediately. Choose between VS Human and VS CPU modes, or sign in to save your progress and access additional features.

# Enjoy playing Chess-Titans and have fun! 🌟🌍🏆


![thank-you](https://github.com/Punith-KK/Chess-Titans/assets/118302022/5a7bb22e-3cc1-4bc6-abeb-80c91273bceb)















