# noclip.website debug scripts list
> **Please read the README.md of this repository if you haven't yet.**

__**This separate list is dedicated to more technical scripts that have little or zero use to the general user who just wants to play around.**__

If you are a developer either trying to make your own scripts like the ones on this repository or contributing to noclip directly, some of these may come in handy for you.

# Contents Index
- [debugJunk](#debugjunk)
  - [hexdump](#hexdump)
  - [interactiveVizSliderSelect](#interactivevizsliderselect)
- [inputManager](#inputmanager)
  - [registerKeyTrigger](#registerkeytrigger)

## debugJunk
This category is for scripts located in the aptly named `main.debugJunk` object on noclip's console.

`main.debugJunk` has multiple functions and not all of them will be here either because I don't know what they do or simply don't think they're relevant to be here, but you're welcome to play around with it on your own.

### hexdump
A fancy hex viewer directly on the console for ArrayBuffer objects.

**Usage:**
```js
main.debugJunk.hexdump(data: ArrayBuffer, start_offset: Number = 0, length: Number = 0x100)
```
- data: the ArrayBuffer to create a hex view of.
- start_offset: (Optional - Default value: `0x0`) the hex offset to start the hex view from.
- length: (Optional - Default value: `0x100`) the amount of bytes to display on the hex view from the start_offset.

​
### interactiveVizSliderSelect
On-screen dynamic slider with selector that takes an array of objects as input and filters them by a boolean property.
> Example usage on the [Display Object Slider](/SCRIPTS.md#display-object-slider) script for SMG1/2

**Usage:**
```js
main.debugJunk.interactiveVizSliderSelect(objects: Array, property: String = 'visible', callback?: Function)
```
- objects: An array of objects to be filtered into the slider.
- property: (Optional - Default value: `'visible'`) The boolean property in the root of the objects of the array to filter the objects with. Only objects whose property is `true` will be put into the slider.
- callback: (Optional - Default value: none) A function to be called when an object is selected on the slider. Receives one parameter with the index of the selected object as a Number.

​

​
## inputManager
This category is for scripts located in the `main.viewer.inputManager` object on noclip's console.

### registerKeyTrigger
Utility for registering a keybind to run some code when pressed on noclip.

> ⚠️ These keybinds are active globally. Keep this in mind before assigning game-specific code to one.

> ⚠️ **If an uncaught error happens in a keybind's code, noclip will freeze and need to be reloaded.**

Multiple functions can be assigned to the same key by simply calling `registerKeyTrigger` on the same key twice with different callbacks.

The execution order of multiple callbacks assigned to the same key will be the same as they were assigned.

**Usage:**
```js
main.viewer.inputManager.registerKeyTrigger(keycode: String, callback: Function)
```
- keycode: The keyboard key to assign as keybind. Example: `KeyG` (Assigns the letter **G** key)
- callback: The function to run when the keybind is pressed.
