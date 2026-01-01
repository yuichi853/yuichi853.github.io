+++
title = "2dgameengine"
author = ["Yuichi Tsunoda"]
date = 2025-12-29T00:00:00+09:00
draft = false
+++

## Libraries and Dependencies {#libraries-and-dependencies}


### Compile and Link {#compile-and-link}

Compile and link are two different stages.


#### Compiler stage {#compiler-stage}

-   compile stage
-   compiler reads your .cpp
-   compiler reads header files (.h)
-   compiler checks:
    -   “Does this function name exist?”
    -   “Are argument types correct?”
    -   “Is the structure defined?”
-   compiler generates .o (object files)


#### Link stage {#link-stage}

-   linker takes object files + binary libraries
-   linker attaches real implementations from .so/.a
-   produces the final executable

So:

| Stage   | Needs headers? | Needs binaries? |
|---------|----------------|-----------------|
| Compile | Yes            | No              |
| Link    | No             | Yes             |


#### Q&amp;A from cource {#q-and-a-from-cource}

Q
: You explain that we install the binaries for e.g. SDL. But why do we then include  **.h files (e.g. SDL.h). Are header files not text files?**

A
: **So, even when we are using binaries (.libs), we still need to add a .h file to our project.**
    Header files (.h) are indeed source-code/text files, but the main goal of a header file is to expose the API of those binaries that we are trying to use.
    For example, when we include the binary of the SDL library, we need to also add the .h files for SDL. These .h files will tell the compiler important things, like what are the function names and the order of the parameters and also their type.
    When we compile our project, the compiler only looks at the header files and does a quick check to see if the function names are correct and the parameters are the ones that the header file expects.
    After the compiler says everything looks good and our calls match what the header file expects, the linker is the one responsible for bundling the implementation/binaries of those functions with our final executable.
    So, that's the reason why we still need .h files. These header files are source code files, but they only tell the compiler what we can call from those binaries. I always like to think that the header file tells the compiler what are the names of the functions we can call, their return type, and also the parameters with their types and the order we need to pass them to the function.
    That's the reason we downloaded both the Windows binaries (.lib) and the header files (.h) from the SDL official website.


### Static vs Dynamic Libraries {#static-vs-dynamic-libraries}


#### 1. Static Libraries (.lib / .a) {#1-dot-static-libraries--dot-lib-dot-a}

-   Library code is copied into the final executable at link time.
-   Each program gets its own private copy of the code.
-   Results in a larger executable and higher overall memory usage.
-   Cannot be shared between multiple running programs.
-   Common file extensions:
    -   Windows: .lib
    -   Linux/Unix: .a


#### 2. Dynamic Libraries (.dll / .so) {#2-dot-dynamic-libraries--dot-dll-dot-so}

-   Library code stays outside the executable and is loaded at runtime.
-   Multiple programs can share the same library in memory.
-   Executable size is smaller and memory use is more efficient.
-   Common file extensions:
    -   Windows: .dll
    -   Linux/Unix: .so


#### Comparison Table {#comparison-table}

| Feature         | Static (.lib/.a)          | Dynamic (.dll/.so)               |
|-----------------|---------------------------|----------------------------------|
| Link time       | Build time                | Run time                         |
| Executable size | Larger                    | Smaller                          |
| Memory usage    | Higher (duplicate copies) | Lower (shared)                   |
| Sharing allowed | No                        | Yes                              |
| Code location   | Inside the executable     | External file loaded dynamically |


#### Key Idea Summary {#key-idea-summary}

-   Static library: code is **copied** into your program.
-   Dynamic library: code is **shared** at runtime among programs.


## Displaying the Game Window {#displaying-the-game-window}


### Game Loop {#game-loop}


#### Game Loop {#game-loop}

1.  Process Input
2.  Update Game
3.  Render (display stuff)

<!--listend-->

```cpp
while (true) {
  game -> processInput();
  game -> update();
  game -> render();
 }
```


#### Game loop on game engine {#game-loop-on-game-engine}

The game engine already provides the core game loop.
When we write scripts to control game objects, the game engine exposes several key functions for us to override.

<!--list-separator-->

-  Three important functions

    1.  `SETUP`
        -   Initializes the game state.
        -   Called **once** at the very beginning.
        -   Used to configure initial positions, colors, speeds, and any starting values.
    2.  `UPDATE`
        -   Updates game objects every frame.
        -   Called **multiple times per second** (depending on the frame rate).
        -   Handles movement, physics, input, and gameplay logic.
    3.  `DRAW` (or `RENDER`)
        -   Renders all game objects to the screen.
        -   Also called **multiple times per second**.
        -   Handles drawing sprites, shapes, and UI elements each frame.

<!--list-separator-->

-  Summary

    -   **SETUP**: runs once → initialize game.
    -   **UPDATE**: runs every frame → update game logic.
    -   **DRAW**: runs every frame → render visuals.


### Game Class {#game-class}

In OOP languages like C++, you can create classes to divide the responsibilities of the process.


#### Include file/folder {#include-file-folder}

When you include the file at the same folder (sibling file), use double quote `"` to import it.
When you include the operating system include folder, use angle bracket `< >` to import it.

```cpp
#include "Game.h"
#include <SDL.h>
```

To use the function from the `Game` class, the only thing you need to  include is the `Game.h` header file.
This is because the compiler only cares about the `function names` and their `parameters`.
The actual implementations of the functions written in `Game.cpp` are not required during complilation; they are added later during the `linking stage`.
At that stage, the linker combines all the function implementations into the final executable.


#### Protection Guard {#protection-guard}

Whenever we declare a header file, we need to add a protection guard.
The protection guard makes the preprocessor include the header file once in the entire project.
In C++ we include a header file all over the place.
So, we need to prevent the preprocessor from copying the same header file multiple times.

```cpp
#pragma once

class Game {
    //...
};
```


#### Store object in the stack {#store-object-in-the-stack}

Since we are not using the `new` keyword to create the object, this object will be stored in the `stack` and will be destroyed when the scope ends.

```cpp
int main(int argc, char* argv[]) {
    // Create the object on stack
    Game game;

    // 'new Game()' dynamically allocates memory in the heap, not the stack
    // This allocation does not end automatically when scope ends.
    Game* game = new Game();
    // So, you must delete it manually
    delete game;

    return 0;
}
```


### Creating an SDL Window {#creating-an-sdl-window}


#### Renderer {#renderer}

A renderer is the object responsible for `drawing` into the window.

The renderer:

-   Handles all drawing commands
    (clear screen, draw textures, draw shapes, copy images, etc.)
-   Sends your drawing to the GPU
-   Controls color, textures, blending, etc.
-   Updates the window when you call `SDL_RenderPresent`

<!--listend-->

```cpp
void Game::Initialize() {
    // Initialize SDL...
    // Initialize Window...

    // Create to renderer to go inside the window
    SDL_Renderer* renderer = SDL_CreateRenderer(window, -1, 0);
}
```


### Fullscreen Window (Fake-Fullscreen) {#fullscreen-window--fake-fullscreen}


#### Display Mode API {#display-mode-api}

SDL uses a C-style API. Instead of returning a full struct, functions like `SDL_GetCurrentDisplayMode` take a `pointer to a struct` and `write data directly into that struct`.

This approach is common in C libraries for several important reasons:

-   C functions can only return one value, so using a struct pointer allows the function to return `multiple pieces of information` at once.
-   Returning large structs by value would require copying memory, which is inefficient. Passing a pointer avoids unnecessary memory operations.
-   SDL is implemented in pure C, so its API follows `traditional C conventions` to maintain compatibility with both C and C++.

After the function call, the struct you provided will be populated with the current display's properties. In particular, `displayMode.w` and `displayMode.h` will contain the monitor’s resolution.

Key points:

-   SDL’s API uses `pass-by-reference` (pointer) style because of C language design.
-   You create an `empty struct` and pass its address; SDL `fills in the fields`.
-   No value copying is involved; the struct is `modified directly` for efficiency.
-   After the function call, you can read the `updated width and height` from it.

<!--listend-->

```cpp
SDL_DisplayMode displayMode;    // Struct to hold display information

SDL_GetCurrentDisplayMode(
    0,               // Display index (0 = primary monitor)
    &displayMode     // Pass pointer so the function can fill the struct
);

// The struct now contains the screen resolution
windowWidth  = displayMode.w;   // Retrieved monitor width
windowHeight = displayMode.h;   // Retrieved monitor height
```


#### Change the video mode {#change-the-video-mode}

Change the video mode to the actual full-screen.

```cpp
SDL_SetWindowFullscreen(window, SDL_WINDOW_FULLSCREEN);
```


### Fake Fullscreen vs Real Fullscreen {#fake-fullscreen-vs-real-fullscreen}


#### Best Solution for Setting the Screen {#best-solution-for-setting-the-screen}

One simple way to run your game in fullscreen mode is to manually specify a fixed resolution and then instruct SDL to switch the window into fullscreen video mode. In this approach, you directly assign the desired width and height to your window variables, and SDL will attempt to scale or adapt the window to match the monitor's fullscreen mode. Different players may have different monitor environments (aspect ratios, resolutions, and scaling factors), so simply switching to fullscreen can cause the visible area of the game to vary from player to player. By defining a fixed rendering resolution yourself, you ensure that the portion of the game world shown on the screen is always consistent regardless of the user’s monitor, because SDL will scale your defined resolution to fit their display.

Below is an example:

```cpp
windowWidth  = 800;   // Desired width for the fullscreen window
windowHeight = 600;   // Desired height for the fullscreen window

// Switch the window into fullscreen mode using the chosen resolution
SDL_SetWindowFullscreen(window, SDL_WINDOW_FULLSCREEN);
```

This approach forces SDL to use your specified resolution when entering fullscreen mode. It is simple and predictable, making it useful when your game is designed for a specific resolution or when you want full control over the rendering dimensions and the exact area of the game world that players see, independent of their monitor setup.


### Understanding the SDL Renderer (GPU Acceleration and VSync) {#understanding-the-sdl-renderer--gpu-acceleration-and-vsync}

It is helpful to understand how SDL handles rendering behind the scenes. SDL is a smart library, and by default it will try to detect whether the system has a dedicated graphics card (GPU) and use it to render objects on the screen. When we create a renderer using `SDL_CreateRenderer`, we can optionally specify flags that tell SDL how we want the rendering to behave.

One important flag is `SDL_RENDERER_ACCELERATED`. This flag instructs SDL to attempt to use hardware acceleration instead of software rendering. In most cases, SDL will choose this automatically, but setting the flag explicitly ensures that SDL will try to find and use the GPU.

```cpp
SDL_CreateRenderer(
    window,
    -1,
    SDL_RENDERER_ACCELERATED
);
```

Another useful flag is `SDL_RENDERER_PRESENTVSYNC`. VSync (vertical synchronization) is a technique that synchronizes the game's frame rendering with the monitor's refresh rate. Enabling VSync helps prevent screen tearing artifacts, because it forces each frame to be presented only when the monitor is ready to refresh. However, enabling VSync will limit your rendering FPS to the monitor's refresh rate. For example, even if your game can render at 3000 FPS, enabling VSync on a 60Hz monitor will cap your rendering FPS at 60. Although the frame rate is limited, you can still update game logic or physics at a higher frequency if needed.

We can combine both GPU acceleration and VSync by using the bitwise OR operator. This tells SDL that we want to enable hardware acceleration and also enable VSync to prevent screen tearing:

```cpp
SDL_CreateRenderer(
    window,
    -1,
    SDL_RENDERER_ACCELERATED | SDL_RENDERER_PRESENTVSYNC
);
```

Using these two flags provides a powerful and flexible configuration: SDL will render using the GPU whenever possible, and VSync will help ensure a smooth visual experience without tearing artifacts.


## Rendering SDL Objects {#rendering-sdl-objects}


### Double-Buffered Renderer {#double-buffered-renderer}

When we call `SDL_RenderPresent(renderer)`, SDL performs a buffer swap that makes the newly rendered frame visible on the screen. Internally, SDL uses a double-buffering system consisting of a `front buffer` and a `back buffer`.

-   The `front buffer` is the image currently displayed on the monitor.
-   The `back buffer` is where SDL draws the next frame while the front buffer is still being shown.

All rendering commands (such as drawing textures, shapes, or clearing the screen) are applied only to the `back buffer`. This keeps the currently visible image stable while the new frame is being prepared in the background.

When the frame is fully drawn, calling `SDL_RenderPresent(renderer)` swaps the two buffers:

1.  The back buffer becomes the new front buffer and is displayed on the screen.
2.  The old front buffer becomes the new back buffer, ready to be drawn on for the next frame.

This double-buffering system prevents flickering and tearing, because partially drawn frames are never shown to the player. Only fully completed frames are displayed after the buffer swap.

```cpp
// Present the renderer to the screen (swap back buffer → front buffer)
SDL_RenderPresent(renderer);
```


### Initialize our game object {#initialize-our-game-object}

`Setup()` is usually a function that is called once, at the beginning of your game. It can be used to initialize our game object values and properties.


### Drawing a PNG Texture {#drawing-a-png-texture}

```cpp
    SDL_Surface* surface = IMG_Load("./assets/images/tank-tiger-right.png");
    SDL_Texture* texture = SDL_CreateTextureFromSurface(renderer, surface);
    SDL_FreeSurface(surface);

    SDL_Rect dstRect = { 10, 10, 32, 32 };
    SDL_RenderCopy(renderer, texture, <srcRect>,<dstRect>);

    SDL_DestroyTexture(texture);
```

This code loads a PNG image from disk, converts it into a format that the GPU can draw, and then renders it on the screen.

```cpp
SDL_Surface* surface = IMG_Load("./assets/images/tank-tiger-right.png");
```

When we call `IMG_Load`, SDL loads the PNG file and places the pixel data into a `surface`. A `surface` is simply raw image data stored in normal RAM. It represents an image that the CPU can access, but <span class="underline">it is not in a format that the GPU can draw efficiently</span>.

```cpp
SDL_Texture* texture = SDL_CreateTextureFromSurface(renderer, surface);
```

To render the image on the screen, we must convert the surface into a `texture`. A `texture` is the GPU-friendly version of the image. Textures live on the graphics card and can be drawn very efficiently by the renderer.

```cpp
SDL_FreeSurface(surface);
```

After converting the image into a texture, the original surface is no longer needed, so we free it to avoid wasting memory.

```cpp
SDL_Rect dstRect = { 10, 10, 32, 32 };
SDL_RenderCopy(renderer, texture, <srcRect>, <dstRect>);
```

Finally, `SDL_RenderCopy` copies the texture onto the current back buffer. <span class="underline">The `srcRect` specifies which part of the texture to use, and the `dstRect` specifies where and how large it should appear on the screen.</span>
In most of the case, we want to use the entire texture. So, If you want to draw the whole image, you can pass `NULL`.

```cpp
SDL_DestroyTexture(texture);
```

After converting an image into a `texture` and drawing it with `SDL_RenderCopy`, we should free the memory used by the texture when we are done with it. SDL textures live on the GPU, which means they consume graphics memory. If we never destroy them, the GPU memory will eventually fill up, causing performance problems or crashes. To release the texture properly, we call `SDL_DestroyTexture`.

This tells SDL to remove the texture from the GPU and free all resources associated with it. It is the GPU equivalent of freeing a surface with `SDL_FreeSurface`. Always make sure to destroy textures once you are done drawing them, especially inside loops or when loading many images.


## Fixing the Game Time Step {#fixing-the-game-time-step}


### Object Movement and Velocity Vectors {#object-movement-and-velocity-vectors}
