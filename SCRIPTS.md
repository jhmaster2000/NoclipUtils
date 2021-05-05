# noclip.website scripts list
> **Please read the README.md of this repository if you haven't yet.**

This list is categorized by which part of the website the scripts are for.

You're welcome to try running game-specific scripts on other games, but they will most likely just error out.

- **[Contents Index](https://github.com/jhmaster2000/NoclipUtils#contents-index)**

## Global Scripts
These scripts (hopefully) work anywhere on the website.

### Teleport Camera
Teleports the camera to the specified coordinates.

**Usage:** `teleport(Number: x, Number: y, Number: z)`
```js
function teleport(x, y, z) {
	if (!x || !y || !z) return console.warn('All three coordinates required.');
	if (isNaN(x) || isNaN(y) || isNaN(z)) return console.warn('Invalid coordinates.');
	
	if (x === null) x = main.viewer.camera.worldMatrix[12];
	if (y === null) y = main.viewer.camera.worldMatrix[13];
	if (z === null) z = main.viewer.camera.worldMatrix[14];
    
    main.viewer.camera.worldMatrix[12] = x;
    main.viewer.camera.worldMatrix[13] = y;
    main.viewer.camera.worldMatrix[14] = z;
    main.viewer.cameraController.forceUpdate = true;
    
    return console.log('Teleported to: ' + x + ' ' + y + ' ' + z);
}
```

​
### Toggle camera upward-axis
Toggles camera upward-axis behavior when turning upside-down/sideways. *Especially useful for the Galaxy games.*
> The `main.viewer.cameraController.useViewUp` code snippet was provided publicly by Jasper on the noclip Discord.

Modes:
- global: The default mode, where the up axis is fixed to the scene's true up axis.
- self: The up axis is relative to the camera's orientation, so turning the camera upside-down makes "down" be your "up" axis.

Hint: Use the keybinds `U` and `O` to tilt the camera on either mode.

**Usage:** `toggleCameraUpAxisMode()`
```js
function toggleCameraUpAxisMode() {
	var useViewUp = main.viewer.cameraController.useViewUp;
	
	if (useViewUp) {
		main.viewer.cameraController.useViewUp = false;
		return 'Camera Z-Axis set to self.';
	}
	
	main.viewer.cameraController.useViewUp = true;
	return 'Camera Z-Axis set to global.';
}
```

​
### Toggle noclip.website Branding
Toggles the bottom-right noclip branding used by Jasper on his Twitter videos.
> The `main.ui.recordingBranding.v()` code snippet was provided publicly by Jasper on the noclip Discord.

**Usage:** `toggleBranding()`
```js
function toggleBranding() {
	if (!main.ui.recordingBranding.activated) {
		main.ui.recordingBranding.v();
		main.ui.recordingBranding.activated = true;
		return 'Branding enabled.';
	}
	
	if (!main.ui.recordingBranding.elem.hidden) {
		main.ui.recordingBranding.elem.hidden = true;
		return 'Branding Disabled.';
	}
	
	main.ui.recordingBranding.elem.hidden = false;
	return 'Branding enabled.';
}
```

​
### Enable Studio Mode
Enables the *Work In Progress* Studio mode for making cinematic cameras.
> This script was provided publicly by Jasper on the noclip Discord.

> ⚠️ This script requires reloading the page to disable.

**Usage:** `enableStudioMode()` or just `main.ui.studioPanel.v()` directly.
```js
function enableStudioMode() {
	main.ui.studioPanel.v();
	return 'Successfully executed.';
}
```

​

​
## Super Mario Galaxy 1/2 Scripts

### Display Object Slider
Displays the object picker slider for all currently visible objects. Click **Select** to disable it.

Sliding the slider will make all objects not covered by the slider not visible until the slider is disabled.

Clicking **Select** will also make the currently selected object on the slider flash for 3 seconds and print the object in the console.
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** `SMG_getObjectSlider()`
```js
function SMG_getObjectSlider() {
  var objectList = main.scene.sceneObjHolder.nameObjHolder.nameObjs;
  main.debugJunk.interactiveVizSliderSelect(objectList, 'visibleScenario');
}
```

​
### Teleport To Object
Teleports the camera to the targetted object.

**Usage:** `SMG_teleportTo(String: objname, Number: index)`
- objname: Must be the object's internal SMG name, for example `AstroCore` for the Comet Observatory's core.
- index: (Optional parameter - Default: 0) If there are multiple instances of an object, such as `Coin`, use this 0-indexed incrementing value to pick between instances.
```js
function SMG_teleportTo(objname, index) {
  var objectList = main.scene.sceneObjHolder.nameObjHolder.nameObjs;
	var i = index;
	if (!index) i = 0;
	
	var obj = objectList.filter((g) => g.name === objname);
	
    if (!obj[i]) return console.warn('Inaccessible object: ' + objname);
    if (!obj[i].translation) return console.warn('Unmovable object: ' + objname);
    
    main.viewer.camera.worldMatrix[12] = obj[i].translation[0];
    main.viewer.camera.worldMatrix[13] = obj[i].translation[1];
    main.viewer.camera.worldMatrix[14] = obj[i].translation[2];
    main.viewer.cameraController.forceUpdate = true;
    
    return console.log('SMG - Teleported to: ' + obj[i].translation[0] + ' ' + obj[i].translation[1] + ' ' + obj[i].translation[2]);
}
```

​
### Warp Object to Camera
Warps (teleports) a targetted object to the current camera position.

**Usage:** `SMG_warpHere(String: objname, Number: index)`
- objname: Must be the object's internal SMG name, for example `AstroCore` for the Comet Observatory's core.
- index: (Optional parameter - Default: 0) If there are multiple instances of an object, such as `Coin`, use this 0-indexed incrementing value to pick between instances.
```js
function SMG_warpHere(objname, index) {
    var objectList = main.scene.sceneObjHolder.nameObjHolder.nameObjs;
    var i = index;
    if (!index) i = 0;
    
    var worldMatrix = main.viewer.camera.worldMatrix;
    var obj = objectList.filter((g) => g.name === objname);
    
    if (!obj[i]) return console.warn('Inaccessible object: ' + objname);
    if (!obj[i].translation) return console.warn('Unmovable object: ' + objname);

    obj[i].translation[0] = worldMatrix[12];
    obj[i].translation[1] = worldMatrix[13];
    obj[i].translation[2] = worldMatrix[14];
    
    return console.log('SMG - Object warped: ' + objname);
}
```

​
### Force All Objects Visible
Forces all objects from every scenario (mission) in the current galaxy to be visible.

This also makes objects that require events to appear become visible without needing to activate said events, since most events aren't possible to activate on noclip.
> ⚠️ This script requires reloading the page to disable.

**Usage:** `SMG_allObjectsVisible()`
```js
function SMG_allObjectsVisible() {
  var objectList = main.scene.sceneObjHolder.nameObjHolder.nameObjs;
	objectList.forEach(o => {
		o.visibleScenario = true;
		o.visibleModel = true;
		o.visibleAlive = true;
	});
	return 'SMG - Forced all objects to be visible.';
}
```

​

​
## Zelda Wind Waker Scripts

### Toggle Forcing Time to Pass Everywhere
Some islands on The Great Sea of Zelda Wind Waker freeze the passing of time when you enter them. This script can override this time freeze and force time to pass even on those islands.
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** Run as shown below to enable forced time passing, change `true` to `false` to disable.
* You may need to wait some time for the time to reach a certain point before it freezes again upon disabling forced time passing depending on what the current time is.
```js
main.scene.globals.g_env_light.forceTimePass = true
```

​

​
## Wii Sports Resort Scripts

### Toggle Vertex Colors
Enables/disables vertex colors on Wii Sports Resort.
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** Run as shown below to disable, change `false` to `true` to re-enable.
```js
main.scene.modelInstances.forEach((v) => v.setVertexColorsEnabled(false));
```
