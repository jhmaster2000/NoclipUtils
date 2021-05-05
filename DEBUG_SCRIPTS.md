# noclip.website debug scripts list
> **Please read the README.md of this repository if you haven't yet.**

__**This separate list is dedicated to more technical scripts that have little or zero use to the general user who just wants to play around.**__

If you are a developer either trying to make your own scripts like the ones on this repository or contributing to noclip directly, some of these may come in handy for you.

# Contents Index
- [debugJunk](https://github.com/jhmaster2000/NoclipUtils/blob/master/DEBUG_SCRIPTS.md#debugjunk)
  - [hexdump](https://github.com/jhmaster2000/NoclipUtils/blob/master/DEBUG_SCRIPTS.md#hexdump)
  - [interactiveVizSliderSelect](https://github.com/jhmaster2000/NoclipUtils/blob/master/DEBUG_SCRIPTS.md#interactivevizsliderselect)

## debugJunk
This category is for scripts located in the aptly named `main.debugJunk` object on noclip's console.

`main.debugJunk` has multiple functions and not all of them will be here either because I don't know what they do or simply don't think they're relevant to be here, but you're welcome to play around with it on your own.

### hexdump
A fancy hex viewer directly on the console for ArrayBuffer objects.

**Usage:**
```js
main.debugJunk.hexdump(ArrayBuffer: data, Number: start_offset, Number: length)
```
- data: the ArrayBuffer to create a hex view of.
- start_offset: (Optional parameter - Default value: `0x0`) the hex offset to start the hex view from.
- length: (Optional parameter - Default value: `0x100`) the amount of bytes to display on the hex view from the start_offset.

​
### interactiveVizSliderSelect
On-screen dynamic slider with selector that takes an array of objects as input and filters them by a boolean property.
> Example usage on the [Display Object Slider](https://github.com/jhmaster2000/NoclipUtils/blob/master/SCRIPTS.md#display-object-slider) for SMG1/2

**Usage:**
```js
main.debugJunk.interactiveVizSliderSelect(Array: objects, String: property, Function: callback)
```
- objects: An array of objects to be filtered into the slider.
- property: The boolean property in the root of the objects of the array to filter the objects with. Only objects whose property is `true` will be put into the slider.
- callback: (Optional parameter) A function to be called when an object is selected on the slider. Receives one parameter with the index of the selected object as a Number.