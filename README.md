# Unreal Engine Projects

**Snake**, a C++ game in Unreal Engine 4.27, plus two Unreal Engine 5.3 course exercises: a brick-breaker built in Blueprints and a level on the Third Person template. Uploaded in May 2024.

| Project | Engine | What it is |
|---|---|---|
| [`Snake/`](Snake) | UE 4.27, C++ | Grid-based Snake with C++ gameplay classes and Blueprint subclasses |
| [`Seminar 8/`](Seminar%208) | UE 5.3, Blueprints | Brick-breaker with lives, a block generator and random bonuses |
| [`Seminar 3/`](Seminar%203) | UE 5.3 | Course exercise on the Third Person template |

<!-- Gameplay GIF: add docs/snake.gif and embed it here -->

## Snake (UE 4.27, C++)

A top-down Snake: the snake moves one cell at a time, grows when its head reaches food, may speed up after eating, and dies when it hits a wall or its own body. Controls: W/A/S/D or a gamepad's left stick; Esc quits.

All gameplay code is in [`Snake/Source/SnakeGame`](Snake/Source/SnakeGame):

| Class | Role |
|---|---|
| `ASnakeBase` | The snake. Spawns and owns its segments (`TArray<ASnakeElementBase*>`), moves, grows and changes speed |
| `ASnakeElementBase` | One body segment: a static mesh with overlap-only collision. The head segment binds `OnComponentBeginOverlap` |
| `IInteractable` | Interface for anything the snake can touch: `Interact(AActor* Interactor, bool bIsHead)` |
| `AFood` | Implements `IInteractable`: when the head touches it, the snake gains a segment and the food is destroyed |
| `ABoundaryWall` | Blocking wall mesh; the head touching it destroys the snake |
| `APlayerPawnBase` | Top-down camera pawn; spawns the snake and turns the `Vertical`/`Horizontal` input axes into directions |
| `AMyHUD` | Draws a centred game-over message on the canvas and switches input to UI mode |
| `ASnakeGameGameModeBase` | Game-only input mode at start; Esc quits the game |

How it works:

- **The actor tick is the game clock.** `SetActorTickInterval(MovementSpeed)` makes each tick one step. On every step each segment moves to where the segment ahead of it was, and the head moves one cell (`ElementSize`). Speeding up divides the tick interval.
- **Collisions go through an interface.** When the head overlaps an actor, `ASnakeBase::SnakeElementOverlap` casts it to `IInteractable` and calls `Interact`, passing whether the head was involved. `AFood` grows the snake; a body segment's `Interact` destroys it, which is how running into yourself ends the game. Walls are handled separately with `IsA(ABoundaryWall::StaticClass())`.
- **No false overlaps while moving.** The head's collision is switched off while the segments shift and switched back on afterwards (`ToggleCollision`).
- **One turn per step.** A direction change sets `bCanChangeDirection = false` until the next move, so two quick key presses cannot fold the snake back into itself, and 180° turns are rejected.
- **Random speed-up.** Each food eaten has a 50% chance to double the snake's speed (`FMath::FRand() <= 0.5f`).
- **Blueprints for data, C++ for logic.** `BP_Snake`, `BP_SnakeElement`, `BP_Food`, `BP_BoundaryWall`, `PB_PlayerPawn` and `BP_GameMode` subclass the C++ classes and set meshes and defaults. `ASnakeBase::GameOver()` and `SetFirstElementType()` (a `BlueprintNativeEvent`) are exposed to Blueprints.

Code comments and the in-game game-over text are in Russian.

**Run:** open `Snake/Snake.uproject` in Unreal Engine 4.27 and let it build the `SnakeGame` module (needs Visual Studio with the "Game development with C++" workload), then press Play. The level is `Content/Maps/LevelMap_1`.

## Seminar 8: brick-breaker (UE 5.3, Blueprints)

A brick-breaker built entirely in Blueprints (`Content/MyGame`):

- `BP_Platform`: the paddle; `BP_Ball` bounces off surfaces using reflection vectors.
- `BP_BlockGenerator` lays out the blocks; `BP_Block_Parent` and `BP_Block_Child` share block behaviour through inheritance.
- `BP_KillZone` destroys what falls into it below the paddle; `BP_HUD` shows the remaining lives.
- `BP_Bonuses` drops a random bonus whose type comes from the `E_BonusType` enum: extra life, extra ball or a bigger paddle.
- Core classes: `BP_MyGameMode`, `BP_MyPawn`, `BP_MyPlayerController` and `BP_MyGameInstance` (enables VSync on start).

**Run:** open `Seminar 8/Seminar 8.uproject` in Unreal Engine 5.3; the level is `MyGame/Levels/Level_1`.

## Seminar 3 (UE 5.3)

A course exercise based on the Third Person template with Starter Content and level-prototyping assets. Open `Seminar 3/Seminar 3.uproject` in Unreal Engine 5.3; the level is `ThirdPerson/Maps/ThirdPersonMap`.

## Repository notes

Engine-generated folders (`Intermediate/`, `Saved/`) are no longer tracked; the editor recreates them. `Content/` stays in the repository because the projects need their assets to open.
