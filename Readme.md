# ⚔️ Console RPG: TCP Multiplayer & MVC Architecture

<p align="center">
  <img src="https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=c-sharp&logoColor=white" alt="C#"/>
  <img src="https://img.shields.io/badge/.NET-512BD4?style=for-the-badge&logo=dotnet&logoColor=white" alt=".NET"/>
  <img src="https://img.shields.io/badge/Architecture-MVC-blue?style=for-the-badge" alt="Architecture: MVC"/>
  <img src="https://img.shields.io/badge/Networking-TCP%20%2F%20JSON-orange?style=for-the-badge" alt="TCP Network"/>
  <img src="https://img.shields.io/badge/Clean%20Code-No%20RTTI-success?style=for-the-badge" alt="Clean Code"/>
</p>
<p align="center">
  <img src="https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExb2tyMGx6cm40eHYyNHZkNHRtaTd5OTU4OXZudmNhNTZwdWtzODk2YSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/l9eJw5IqiSDwVzb4bh/giphy.gif" width="400px" />
</p>

## 📖 Overview

This project represents a **scalable, real-time client-server system** for multi-user interaction, implemented in C#. Throughout its 7-stage evolution, the architecture transitioned from a monolithic console-based game engine into a **distributed system** featuring an **authoritative server** and support for up to 9 concurrent network clients. The development focused on strict **MVC-based decomposition**, ensuring complete decoupling of business logic from presentation and networking layers.

The core engineering constraint of this project was absolute adherence to **Object-Oriented Programming (OOP) principles**. The codebase strictly avoids **Run-Time Type Information (RTTI)** such as `is`, `as`, `typeof`, and type-identifying `enums`. Instead, all logic is driven by **clean polymorphism** and classical **Gang of Four (GoF) design patterns**.

## 🧪 Automated Testing

The solution includes an **xUnit** test project covering core domain behavior:

- weapon decorators and stacked modifiers;
- one-handed and two-handed equipment rules;
- Visitor-based combat calculations and enemy armor;
- player movement and collision with walls;
- currency pickup and cell rendering priority.

Run the complete test suite locally:

```bash
dotnet test ConsoleRPG_1.sln
```

The GitHub Actions workflow in `.github/workflows/tests.yml` restores, builds, and tests the solution automatically on every push and pull request to `main` or `master`.




## 🏗️ Architectural Highlights

* **Model (Source of Truth):** Encapsulates the entire game state (map grid, player stats, NPC behavior, item locations). Entirely decoupled from the console and network layers.
* **View:** Implements separate renderers for the local console and remote network clients (via JSON payloads).
* **Controller:** Validates player input and routes actions to the Model. Each client session is handled by an isolated controller instance.
* **Concurrency & Networking:** Implemented using `TcpListener` and `TcpClient`. Thread safety is ensured through `lock` mechanisms and concurrent collections, allowing the server to handle multiple simultaneous TCP requests without race conditions.


## 🧩 Applied Design Patterns

| Design Pattern | Implementation in Project | Solved Engineering Problem |
| :--- | :--- | :--- |
| **Observer** | Sound Propagation & Herd AI | Allowed enemies to react to noise and intra-species deaths without tight coupling. Implemented *without* the C# `event` keyword to demonstrate deep OOP understanding. |
| **Composite** | Slotted Items & Sockets | Handled complex inventory math recursively (e.g., calculating stats for a sword holding a socket, which holds another socket and a magic stone). |
| **Decorator** | Dynamic Weapon Modifiers | Applied infinite stackable modifiers (e.g., "Unlucky", "Strong") to weapons during world generation without modifying base classes. |
| **Strategy / Builder** | Procedural Dungeon Generation | Enabled modular creation of dungeons based on specific configuration "Themes" (e.g., corridors, rooms, artifact placements). |
| **State** | Reactive Enemy AI | Allowed NPCs to dynamically switch behaviors (Follow, Flee, Ignore) based on a Sight > Sound priority hierarchy and HP thresholds. |
| **Singleton** | Configuration Manager / Logger | Ensures a single point of access to the external configuration files and global event logging stream across all modules. |
| **Director** | Dungeon Generation Control | Orchestrates the sequence of building steps (empty, corridors, central rooms, loot) defined in the Builder interface to produce consistent dungeon layouts. |
| **Visitor** | Combat & Statistic Calculation | Decouples the operation of calculating combat modifiers (or stat propagation) from the item hierarchy, allowing new operations without modifying Item classes. |


---

## 🚀 Key Features by Development Stage
<summary><strong>Stages 1–2: Engine & Procedural Generation</strong></summary>
Implemented a grid-based engine with modular dungeon generation using strategy-based building blocks (halls, rooms, artifacts).


```mermaid
classDiagram
    direction TB

    %% --- ENGINE & CORE ---
    class GameEngine {
        -Map map
        -Player player
        -Renderer renderer
        -bool isRunning
        -bool isInventoryMode
        -int invCursor
        +Run() void
        -HandleInput() void
        -HandleMapInput(ConsoleKey key) void
        -HandleInventoryInput(ConsoleKey key) void
    }

    class Renderer {
        -Map map
        -Player player
        +DrawFrame(bool isInventoryMode, int invCursor) void
        -GetLine(int lineIndex, string floorInfo, bool isInventoryMode, int invCursor) string
    }

    %% --- WORLD ---
    class Map {
        +int Width
        +int Height
        -Cell[,] grid
        +InitializeMap() void
        +GetCell(int i, int j) Cell
    }

    class Cell {
        +Terrain Terrain
        +List~Item~ Items
        +GetDrawSymbol() char
    }

    class Terrain {
        <<abstract>>
        +char Symbol*
        +IsPassable() bool*
    }

    class Floor {
        +char Symbol
        +IsPassable() bool
    }

    class Wall {
        +char Symbol
        +IsPassable() bool
    }

    %% --- ENTITIES ---
    class Player {
        +int X
        +int Y
        +int Strength
        +int Dexterity
        +int Health
        +int Luck
        +int Aggression
        +int Wisdom
        +int Coins
        +int Gold
        +Item? LeftHand
        +Item? RightHand
        +List~Item~ Inventory
        +string LogMessage
        +UnequipLeft() void
        +UnequipRight() void
        +GetTotalDamage() int
        +Move(int dx, int dy, Map map) void
        +PickUp(Map map) void
        +DropItem(Item item, Map map) void
    }

    %% --- ITEM BASE CLASSES ---
    class Item {
        <<abstract>>
        +char Symbol*
        +string Name*
        +GetDamage() int
        +EquipLeft(Player p) void
        +EquipRight(Player p) void
        +UnEquip(Player p) void
        +Drop(Player p, Cell cell) void
        +PickUp(Player p, Cell cell) void
    }

    class Weapon {
        <<abstract>>
        +int damage
        +GetDamage() int
    }

    class OneHandedWeapon {
        <<abstract>>
        +EquipLeft(Player p) void
        +EquipRight(Player p) void
        +UnEquip(Player p) void
    }

    class TwoHandedWeapon {
        <<abstract>>
        -EquipBoth(Player p) void
        +EquipLeft(Player p) void
        +EquipRight(Player p) void
        +UnEquip(Player p) void
    }

    class Currency {
        <<abstract>>
        +ApplyToWallet(Player p)*
        +PickUp(Player p, Cell cell) void
    }

    %% --- SPECIFIC ITEMS ---
    class Sword {
        +char Symbol
        +string Name
    }
    class Axe {
        +char Symbol
        +string Name
    }
    class Greatsword {
        +char Symbol
        +string Name
    }
    class Bone {
        +char Symbol
        +string Name
    }
    class Mug {
        +char Symbol
        +string Name
    }
    class Rope {
        +char Symbol
        +string Name
    }
    class Coin {
        +char Symbol
        +string Name
        +ApplyToWallet(Player p) void
    }
    class Gold {
        +char Symbol
        +string Name
        +ApplyToWallet(Player p) void
    }

    %% --- RELATIONSHIPS ---
    GameEngine *-- Map : "1 to 1"
    GameEngine *-- Player : "1 to 1"
    GameEngine *-- Renderer : "1 to 1"
    
    Renderer o-- Map : "Uses"
    Renderer o-- Player : "Uses"
    
    Map "1" *-- "many" Cell : "Composition"
    Cell "1" o-- "many" Item : "Aggregation"
    Cell --> Terrain : "Holds"
    
    Terrain <|-- Floor : "Inherits"
    Terrain <|-- Wall : "Inherits"
    
    Player "1" o-- "many" Item : "Inventory / Hands"

    Item <|-- Weapon
    Item <|-- Currency
    Item <|-- Bone
    Item <|-- Mug
    Item <|-- Rope

    Weapon <|-- OneHandedWeapon
    Weapon <|-- TwoHandedWeapon
    
    OneHandedWeapon <|-- Sword
    OneHandedWeapon <|-- Axe
    TwoHandedWeapon <|-- Greatsword

    Currency <|-- Coin
    Currency <|-- Gold
```

<summary><strong>Stage 3: Polymorphic Combat System</strong></summary>

*  **Decorator Pattern for Item Modifiers:** Weapons and items can receive infinite, stackable modifiers (e.g., "+5 Damage", "-5 Luck") upon generation. This was achieved by implementing the Decorator pattern, where modifier classes wrap the base item. The base item classes remain completely uncoupled and oblivious to their enhancements, dynamically calculating compounded names (e.g., *"Sword (Unlucky) (Strong)"*) and stats at runtime.
* **Visitor / Double Dispatch Combat Resolution:** Introduced interactive enemies with Armor and HP. Weapons were classified into three polymorphic categories (*Heavy, Light, Magic*), and players were given three distinct attack styles (*Normal, Stealth, Magic*). The architecture leverages a Strategy/Visitor hybrid pattern. Attack styles act as visitors, interacting directly with the specific type of weapon equipped to calculate the final math.

```mermaid
classDiagram
    %% ==========================================
    %% DECORATOR PATTERN (Item Modifiers)
    %% ==========================================
    class Item {
        <<abstract>>
        +GetName() String*
        +GetDamage() int*
        +Accept(visitor: IAttackVisitor, player: Player) CombatResult*
    }

    class Weapon {
        <<abstract>>
        #baseDamage: int
        +GetDamage() int
    }
    Item <|-- Weapon

    class HeavyWeapon {
        +Accept(visitor: IAttackVisitor, player: Player) CombatResult
    }
    class LightWeapon {
        +Accept(visitor: IAttackVisitor, player: Player) CombatResult
    }
    class MagicWeapon {
        +Accept(visitor: IAttackVisitor, player: Player) CombatResult
    }
    Weapon <|-- HeavyWeapon
    Weapon <|-- LightWeapon
    Weapon <|-- MagicWeapon

    class ItemDecorator {
        <<abstract>>
        #_decoratedItem: Item
        +GetName() String
        +GetDamage() int
        +Accept(visitor: IAttackVisitor, player: Player) CombatResult
    }
    Item <|-- ItemDecorator
    ItemDecorator --> Item : wraps (composition)

    class StrongDecorator {
        +GetDamage() int
        +GetName() String
    }
    class UnluckyDecorator {
        +GetName() String
    }
    ItemDecorator <|-- StrongDecorator
    ItemDecorator <|-- UnluckyDecorator

    %% ==========================================
    %% VISITOR PATTERN (Attack and Combat Strategies)
    %% ==========================================
    class IAttackVisitor {
        <<interface>>
        +VisitHeavy(weapon: HeavyWeapon, player: Player) CombatResult
        +VisitLight(weapon: LightWeapon, player: Player) CombatResult
        +VisitMagic(weapon: MagicWeapon, player: Player) CombatResult
        +VisitRegularItem(item: Item, player: Player) CombatResult
    }

    class NormalAttack {
        +VisitHeavy(weapon: HeavyWeapon, player: Player) CombatResult
        +VisitLight(weapon: LightWeapon, player: Player) CombatResult
        +VisitMagic(weapon: MagicWeapon, player: Player) CombatResult
        +VisitRegularItem(item: Item, player: Player) CombatResult
    }
    class StealthAttack {
        +VisitHeavy(weapon: HeavyWeapon, player: Player) CombatResult
        +VisitLight(weapon: LightWeapon, player: Player) CombatResult
        +VisitMagic(weapon: MagicWeapon, player: Player) CombatResult
        +VisitRegularItem(item: Item, player: Player) CombatResult
    }
    class MagicAttack {
        +VisitHeavy(weapon: HeavyWeapon, player: Player) CombatResult
        +VisitLight(weapon: LightWeapon, player: Player) CombatResult
        +VisitMagic(weapon: MagicWeapon, player: Player) CombatResult
        +VisitRegularItem(item: Item, player: Player) CombatResult
    }
    IAttackVisitor <|.. NormalAttack
    IAttackVisitor <|.. StealthAttack
    IAttackVisitor <|.. MagicAttack

    %% ==========================================
    %% ENTITIES AND GAMEPLAY SYSTEM
    %% ==========================================
    class Player {
        +Strength: int
        +Agility: int
        +Wisdom: int
        +Luck: int
        +Health: int
        -equippedItem: Item
        +GetEquippedItem() Item
    }
    class Enemy {
        +Health: int
        +AttackValue: int
        +Armor: int
        +TakeDamage(amount: int)
    }
    class CombatResult {
        +DamageDealt: int
        +DefenseValue: int
    }

    CombatResult <-- IAttackVisitor : creates
    Player --> Item : owns and equips
    HeavyWeapon ..> IAttackVisitor : passes itself (this)
    LightWeapon ..> IAttackVisitor : passes itself (this)
    MagicWeapon ..> IAttackVisitor : passes itself (this)

```

<summary><strong>Stage 4: Configuration & Event Logging</strong></summary>
System initialization via external JSON/INI files. Implemented a thread-safe event log capturing all critical game state changes.
  
```mermaid
 classDiagram
    %% ==========================================
    %% KONFIGURACJA (Wymaganie: wczytywanie pliku)
    %% ==========================================
    class GameConfig {
        +String PlayerName
        +String Theme
        +String FileName
    }
    class ConfigManager {
        +LoadConfig() GameConfig$
    }
    ConfigManager ..> GameConfig : tworzy

    %% ==========================================
    %% DZIENNIK ZDARZEŃ (Wzorzec: Singleton + Interfejs)
    %% ==========================================
    class ILogger {
        <<interface>>
        +Log(message: String)
        +GetAllLogs() List~String~
        +GetLastLog() String
        +GetLogFilePath() String
    }
    
    class GameLogger {
        -GameLogger _instance$
        +CurrentPlayerId int
        +GetInstance() GameLogger$
        +Log(message: String)
        +ExtractMergedLogs(id: int)
    }
    
    class FileAndMemoryLogger {
        -_logs: List~String~
        -_filePath: String
        +Log(message: String)
    }
    
    class FileLogger {
        -_filePath: String
        +Log(message: String)
    }

    ILogger <|.. GameLogger
    ILogger <|.. FileAndMemoryLogger
    ILogger <|.. FileLogger

    %% ==========================================
    %% MOTYWY LOCHU (Wzorzec: Abstract Factory)
    %% ==========================================
    class IDungeonThemeFactory {
        <<interface>>
        +GetGreetingMessage() String
        +GetSize() Tuple~int, int~
        +CreateGenerationStrategy() IEnumerable~IDungeonStep~
        +CreateItemPool() IEnumerable~Func~Item~~
        +CreateArtifact() Item
        +CreateEnemyPool() IEnumerable~SpeciesSpawnDefinition~
    }

    class VaultThemeFactory
    class LibraryThemeFactory
    class SciFiThemeFactory

    IDungeonThemeFactory <|.. VaultThemeFactory
    IDungeonThemeFactory <|.. LibraryThemeFactory
    IDungeonThemeFactory <|.. SciFiThemeFactory

    class ThemeSelector {
        +SelectTheme(themeName: String) IDungeonThemeFactory$
    }
    
    class DungeonDirector {
        +CreateThemeMap(theme: IDungeonThemeFactory) Map
    }

    ThemeSelector ..> IDungeonThemeFactory : wybiera
    DungeonDirector ..> IDungeonThemeFactory : używa do budowy
```

<summary><strong>Stage 5 : AI Behavior & Acoustic Pathfinding</strong></summary>
  
This stage introduces reactive environmental systems and decoupled communication using a custom **Observer** pattern (strictly avoiding the C# event).
* Herd AI (Collective Behavior): Enemies are grouped into species subjects. When an entity dies, it broadcasts its death to its kin before unregistering (e.g., Goblins lose stats, aggressive Skeletons gain stats).
* Sound Propagation: Picking up weapons emits noise events based on weight class (Heavy = far, Light = close). The player broadcasts the sound and enemies calculate if the sound reaches them using grid pathfinding taking in considiration walls.
* Memory Safety: Strict decoupling ensures that dying enemies explicitly unsubscribe from all observable lists to prevent memory leaks and zombie references.

```mermaid
classDiagram
    class Map {
        +EventManagerSound soundManager
    }

    class EventManagerSound {
        -List~IEventListenerSound~ _listeners
        +Subscribe(listener)
        +Unsubscribe(listener)
        +Notify(x, y, range, map)
    }

    class EventManagerDeath {
        -string _speciesName
        -List~IEventListenerDeath~ _listeners
        +Subscribe(listener)
        +Notify()
    }

    class IEventListenerSound {
        <<interface>>
        +SoundProduced(dist, sourceX, sourceY)
    }

    class IEventListenerDeath {
        <<interface>>
        +MemberDied(name)
    }

    class Player {
        +SoundProduced(dist, sourceX, sourceY)
    }

    class Enemy {
        <<abstract>>
        +EventManagerDeath group
        +EventManagerSound enemiesManagerSound
        +MemberDied(name)
        +SoundProduced(dist, sourceX, sourceY)
    }

    class Zombie
    class Skeleton
    class Goblin

    Map "1" *-- "1" EventManagerSound
    EventManagerSound o-- "*" IEventListenerSound
    EventManagerDeath o-- "*" IEventListenerDeath

    IEventListenerSound <|.. Player
    IEventListenerSound <|.. Enemy
    IEventListenerDeath <|.. Enemy

    Enemy <|-- Zombie
    Enemy <|-- Skeleton
    Enemy <|-- Goblin
    
    Enemy --> EventManagerDeath : triggers
```

<summary><strong>Stage 6: TCP Multiplayer & MVC Architecture </strong></summary>
  
Transitions the monolithic engine into a networked multiplayer game (up to 9 concurrent players) using strict MVC decoupling and TCP/JSON communication.
* **MVC Refactoring**: 
  * Model (Source of Truth): Encapsulates all game state/logic. Zero references to console or network.
  * View: Purely renders received JSON data to the console. Zero game logic.
  * Controller: Validates local inputs and routes actions to the Model.
* **Authoritative Server**: Uses TcpListener with multi-threading (one Task/Thread per client). Safely processes concurrent JSON inputs using lock and broadcasts state updates to all clients.
* **Remote Clients**: Use TcpClient to send local keystrokes as JSON to the server and strictly render the authoritative state received.

```mermaid
classDiagram
    %% ==========================================
    %% MODEL LAYER (ConsoleRPG.Engine)
    %% ==========================================
    namespace Model_Layer_ConsoleRPG_Engine {
        class GameEngine {
            +Map Map
            +Dictionary~int,Player~ Players
            +bool IsServer
            +ExecuteActionForPlayer(int id, ConsoleKey key)
            +SyncStateFromDto(GameUpdateDto dto)
            +ProcessEnemyTurn()
        }
        class Map {
            +Cell[][] Grid
            +SoundManager SoundManager
            +GetCell(int x, int y)
        }
        class Player {
            +int Id
            +int X
            +int Y
            +int Health
            +Move(int dx, int dy)
            +TakeDamage(int amount)
        }
    }

    %% ==========================================
    %% VIEW LAYER (ConsoleRPG.MVC.View)
    %% ==========================================
    namespace View_Layer_ConsoleRPG_View {
        class IGameView {
            <<interface>>
            +Render(GameEngine engine)
            +DisplayLog(string message)
        }
        class ConsoleView {
            +Render(GameEngine engine)
            -DrawMap(Map map)
            -DrawUI(Dictionary players)
        }
    }

    %% ==========================================
    %% CONTROLLER LAYER (ConsoleRPG.MVC.Controller)
    %% ==========================================
    namespace Controller_Layer_ConsoleRPG_Controller {
        class ClientController {
            -GameClient _client
            -IGameView _view
            +RunLoopAsync()
            -HandleLocalInput(ConsoleKey key)
        }
    }

    %% ==========================================
    %% NETWORK LAYER (ConsoleRPG.Networking)
    %% ==========================================
    namespace Network_Layer_ConsoleRPG_Networking {
        class GameServer {
            -TcpListener _listener
            -GameEngine _authoritativeEngine
            -Dictionary~int,TcpClient~ _connectedClients
            +StartServer()
            +BroadcastState()
            -HandleClientTrafficAsync(int playerId, TcpClient client)
        }
        class GameClient {
            -TcpClient _socket
            +GameEngine LocalEngine
            +bool IsActive
            +RunClientAsync()
            +SendInput(ConsoleKey key)
            -ReceiveUpdatesAsync()
        }
    }

    GameEngine "1" *-- "1" Map : Contains & Controls
    GameEngine "1" *-- "0..*" Player : Manages Entities

    ConsoleView ..|> IGameView : Implements
 
    ConsoleView ..> GameEngine : Reads State to Draw

    ClientController "1" --> "1" IGameView : Holds & Triggers Render
    ClientController "1" --> "1" GameClient : Sends Decoded Input

    GameServer "1" *-- "1" GameEngine : Hosts Authoritative Model
    
    GameClient "1" *-- "1" GameEngine : Hosts Local Synced Model
```


## 🔧 Technical Setup

**System Requirements:**
* .NET SDK 8.0+

**Running the System:**

1. **Start the Server:**
   ```bash
   dotnet run -- --server 5555
   ```
2. **Connect a Client:**
   ```bash
   dotnet run -- --client 127.0.0.1:5555
   ```
---
Developed as a project for Object-Oriented Programming (Warsaw University of Technology).
