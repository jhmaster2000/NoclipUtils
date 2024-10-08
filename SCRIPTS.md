# noclip.website scripts list
> **Please read the README.md of this repository if you haven't yet.**

This list is categorized by which part of the website the scripts are for.

You're welcome to try running game-specific scripts on other games, but they will most likely just error out.

- **[Contents Index](/README.md#contents-index)**

## Global Scripts
These scripts (hopefully) work anywhere on the website.

### Force V-Sync Framerate
On some systems/browsers configurations noclip might run the GPU unbounded, resulting in unreasonably and unnecessarily high GPU usage. This script aims to fix that by allowing you to force the website renderer to V-Sync to a specified framerate desired, such as 60.
> **Note:** Noclip's FPS indicator in the Statistics panel will not match any configured FPS values of this kind, see [Calculate Effective Rendered FPS](#calculate-effective-rendered-fps) script for details.

**Usage:** Change `60` below to the desired target FPS.
- Set it to `0` to disable forced V-Sync override.
- Set it to `Infinity` to force uncapped FPS (even if you had native V-Sync!).
> [!IMPORTANT]
> This script is a **hack**, as such it is inaccurate and the inputted VSYNC_FPS value is a *rough estimate of a target FPS*, NOT the real FPS value that will be reached, the real FPS will generally be lower than the VSYNC_FPS value due to frames lost to browser timing inaccuracies, the degree of how bad these inaccuracies are will vary by browser, OS and game/map chosen on noclip (based on how heavy it is to render), and increasingly worsen the higher VSYNC_FPS is. See [Calculate Effective Rendered FPS](#calculate-effective-rendered-fps) for a way to check your current real FPS value.

```js
globalThis.VSYNC_FPS = 60
```
```js
const nativeReqAnimFrame = requestAnimationFrame;
const nativeCancelAnimFrame = cancelAnimationFrame;
requestAnimationFrame = function requestAnimationFrame(callback) {
    if (!globalThis.VSYNC_FPS) return nativeReqAnimFrame(callback);
    const fps = globalThis.VSYNC_FPS;
    const mspf = 1000 / fps;
    return setTimeout(() => callback(document.timeline.currentTime), mspf);
}
cancelAnimationFrame = function cancelAnimationFrame(id) {
    if (!globalThis.VSYNC_FPS) return nativeCancelAnimFrame(id);
    clearTimeout(id);
}
```

### Calculate Effective Rendered FPS
Noclip's FPS indicator in the Statistics panel is a CPU frames unit, not rendered frames unit, this script can calculate the real rendered FPS for your current environment.

**Usage:**
> [!NOTE]
> The default script behavior is designed as a visual feedback in the console and does not actually *return* the FPS value, for advanced scripting purposes use the `await calcFPS(0)` call format to disable logging and obtain the FPS value returned as a number.

```js
calcFPS()
```
```js
function calcFPS(log = true) {
    const promise = new Promise((resolve) => {
        log && console.log('Calculating effective FPS... (This may take a second)');
        const runs = 60, start = performance.now();
        let frame = runs;
        (function cb() {
            if (frame--) return void requestAnimationFrame(cb);
            const fps = Math.round(1000 * runs / (performance.now() - start));
            log && console.log(`Calculated: ~${fps} frames per second`);
            resolve(fps);
        })();
    });
    return log ? undefined : promise;
}
```

### Teleport Cameras
A group of scripts for teleporting the different camera types to specified coordinates.

#### WASD Camera
Teleports the WASD camera to the specified coordinates.

**Usage:**
```js
teleport(x: Number, y: Number, z: Number) => void
```
```js
function teleport(x, y, z) {
	const wmtx = main.viewer.camera.worldMatrix;
    	x ??= wmtx[12]; y ??= wmtx[13]; z ??= wmtx[14];
        x ||= 0; y ||= 0; z ||= 0;
	wmtx[12] = x;
	wmtx[13] = y;
	wmtx[14] = z;
	main.viewer.cameraController.forceUpdate = true;
	console.log('Teleported to: ' + x + ' ' + y + ' ' + z);
}
```

#### Orbit Camera
Teleports the orbit camera to the specified coordinates around the origin point.

If `absolute` is set to `true` (default), teleports to the exact XYZ given regardless of the origin point.

If `absolute` is set to `false`, teleports to the XYZ relative to the origin point.

**Usage:**
```js
teleportOrbit(x: Number, y: Number, z: Number, absolute?: Boolean) => void
```
```js
function teleportOrbit(x, y, z, absolute = true) {
    	const [tX, tY, tZ] = main.viewer.cameraController.translation;
    	const wmtx = main.viewer.camera.worldMatrix;
    	x ??= wmtx[12]; y ??= wmtx[13]; z ??= wmtx[14];
        x ||= 0; y ||= 0; z ||= 0;
        if (absolute) { x -= tX; y -= tY; z -= tZ; }
    	const oZ = Math.hypot(Math.hypot(x, z), y);
    	const oY =  Math.acos(y / oZ * -1);
    	const oXtmp = x / oZ / Math.sin(oY);
    	const XnegZpos = x < 0 && z >= 0;
    	const XposZpos = x >= 0 && z >= 0;
    	const oX = XnegZpos ? Math.acos(oXtmp * -1) * -1 : (XposZpos ? Math.acos(oXtmp) - Math.PI : Math.acos(oXtmp * -1));
    	main.viewer.cameraController.x = oX || 0;
    	main.viewer.cameraController.y = oY || 0;
    	main.viewer.cameraController.zTarget = -Math.abs(oZ) || -10;
	console.log('Teleported to: ' + x + ' ' + y + ' ' + z);
}
```

#### Orbit Camera Origin
Teleports the orbit camera's origin point to the specified coordinates.

**Usage:**
```js
teleportOrigin(x: Number, y: Number, z: Number) => void
```
```js
function teleportOrigin(x, y, z) {
    const camCtrl = main.viewer.cameraController;
    x ??= camCtrl.translation[0];
    y ??= camCtrl.translation[1];
    z ??= camCtrl.translation[2];
    x ||= 0; y ||= 0; z ||= 0;
    camCtrl.translation[0] = x;
    camCtrl.translation[1] = y;
    camCtrl.translation[2] = z;
    console.log('Teleported orbit origin to: ' + x + ' ' + y + ' ' + z);
}
```

#### Ortho Camera
Teleports the ortho camera to the specified coordinates.
> ⚠️ This script only works as expected with the ortho camera facing directly downwards (Use `Numpad 8` to force it to face straight down).

**Usage:**
```js
teleportOrtho(x: Number, y: Number, z: Number) => void
```
```js
function teleportOrtho(x, y, z) {
    const camCtrl = main.viewer.cameraController;
    x ??= camCtrl.translation[0];
    y ??= camCtrl.zTarget;
    z ??= camCtrl.translation[2];
    x ||= 0; z ||= 0; y = -Math.abs(y) || -10;
    camCtrl.translation[0] = x;
    camCtrl.translation[2] = z;
    camCtrl.zTarget = y;
    console.log('Teleported to: ' + x + ' ' + y + ' ' + z);
}
```

​
### Toggle Camera Upward-Axis
Toggles camera upward-axis behavior when turning upside-down/sideways. *Especially useful for the Galaxy games.*
> The `main.viewer.cameraController.useViewUp` code snippet was provided publicly by Jasper on the noclip Discord.

Modes:
- global: The default mode, where the up axis is fixed to the scene's true up axis.
- self: The up axis is relative to the camera's orientation, so turning the camera upside-down makes "down" be your "up" axis.

Hint: Use the keybinds `U` and `O` to tilt the camera on either mode.

**Usage:**
```js
toggleCameraUpAxisMode() => void
```
```js
function toggleCameraUpAxisMode() {
	let useViewUp = main.viewer.cameraController.useViewUp;
	if (useViewUp) {
		main.viewer.cameraController.useViewUp = false;
		return 'Camera Z-Axis set to self.';
	}
	main.viewer.cameraController.useViewUp = true;
	return 'Camera Z-Axis set to global.';
}
```

​
### Toggle Pointer Lock
Enables/disables noclip locking and hiding your pointer (mouse cursor) when dragging the camera around.
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** Run as shown below to disable pointer lock, change `false` to `true` to re-enable it.
```js
main.viewer.inputManager.usePointerLock = false
```

​
### Change Game Time Speed Scale
Changes the general speed scale of everything being rendered.
> This script was provided publicly by Jasper on the noclip Discord.

> ⚠️ This script may not work with some games.

> ⚠️ **Setting any invalid value will freeze noclip until a page refresh.**

**Usage:** Change `1` below to the number you want the time speed multiplied by.
- The default value for all games is `1`
- Setting this value to `0` will 'pause' the game's time
- Some games may intentionally set a max value internally due to technical limitations
  - Setting values higher than the internal max of a game will just internally round it down to the max

```js
main.sceneTimeScale = 1
```

​
### Change Orthographic Camera Near and Far Planes
Changes the near and far planes used by the orthographic camera.

*If you don't know what near and far planes are, you probably shouldn't try using this script.*
> This script was provided publicly by Jasper on the noclip Discord.

> ⚠️ This script only has effect with the orthographic camera in use.

**Usage:** Run as below, changing the numbers to the desired new value.
- The numbers shown below are the default values.

```js
main.viewer.cameraController.nearPlane = -100000
main.viewer.cameraController.farPlane = 100000
```

​
### Toggle noclip.website Branding
Toggles the bottom-right noclip branding used by Jasper on his Twitter videos.
> The `main.ui.recordingBranding.v()` code snippet was provided publicly by Jasper on the noclip Discord.

**Usage:**
```js
toggleBranding() => void
```
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
### Export Local Savestates and Studio Mode Data
Exports locally created savestates and studio mode animations to a downloaded `.nclsp` plaintext file.

*This script serves as an alternative to pressing `Numpad 3` for those without a numpad.*
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** Run as shown below.
```js
main._exportSaveData()
```

​
### Extra Camera Settings
Batch of simple one-line scripts to edit camera settings.
> All of these scripts were provided publicly by Jasper on the noclip Discord.

#### Change Mouse Panning Sensitivity
Changes how fast the camera looks around when panning.

**Usage:** Run as shown below. The default value is `500`.
- Higher values mean lower sensitivity. `Infinity` is valid and makes mouse panning impossible.
- Negative values are valid and invert the panning axes.
- ⚠️ Setting `0` or other invalid values will freeze the camera irreversibly until a page refresh.

```js
main.viewer.cameraController.mouseLookSpeed = 500
```

#### Change Camera Panning Speed Smoothness & Slipperiness
Changes smoothness & slipperiness of the camera when panning to look around.

**Usage:** Run as shown below. The default value is `0` (No smoothing/slipperiness)
- The two values should be equal for the best results, but you can modify the script to make them different if you want.
- Values around `0.9...` give the camera panning the "cinematic" feel.
- Value `1` makes the camera retain panning speed until manually affected.
- Values above `1` will make the camera panning build up speed rather than slowing down.
- Negative values are extremely buggy and shouldn't be used.
- ⚠️ Negative values or values above `1` can potentially cause a blackscreen until a page refresh.

```js
let t_mLDv = 0
main.viewer.cameraController.mouseLookDragFast = t_mLDv;
main.viewer.cameraController.mouseLookDragSlow = t_mLDv;
```

#### Change Camera Movement Slipperiness
Changes how fast the camera fully stops after you stop moving around.

**Usage:** Run as shown below. The default value is `0.8`.
- Value `0` makes full stop instant.
- Values around `0.9...` give the camera the "slippery ice" feel.
- Value `1` makes the camera retain it's speed forever until manually affected.
- Values above `1` will make the camera build up speed rather than slowing down.
- Negative values are extremely buggy and shouldn't be used.
- ⚠️ Negative values or values above `1` can potentially cause a blackscreen until a page refresh.

```js
main.viewer.cameraController.keyMoveDrag = 0.8
```

​

​
## Super Mario Galaxy 1/2 Scripts

### Display Object Slider
Displays the object picker slider for all currently visible objects. Click **Select** to disable it.

Sliding the slider will make all objects not covered by the slider not visible until the slider is disabled.

Clicking **Select** will also make the currently selected object on the slider flash for 3 seconds and print the object in the console.
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:**
```js
SMG_getObjectSlider() => void
```
```js
function SMG_getObjectSlider() {
	let objectList = main.scene.sceneObjHolder.nameObjHolder.nameObjs;
	main.debugJunk.interactiveVizSliderSelect(objectList, 'visibleScenario');
}
```

​
### Teleport To Object
Teleports the camera to the targetted object.

**Usage:**
```js
SMG_teleportTo(objname: String, index: Number = 0) => void
```
- objname: Must be the object's internal SMG name, for example `AstroCore` for the Comet Observatory's core.
- index: (Optional - Default: 0) If there are multiple instances of an object, such as `Coin`, use this 0-indexed incrementing value to pick between instances.

```js
function SMG_teleportTo(objname, index) {
	let objectList = main.scene.sceneObjHolder.nameObjHolder.nameObjs;
	let i = index;
	if (!index) i = 0;
	let obj = objectList.filter((g) => g.name === objname);
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

**Usage:**
```js
SMG_warpHere(objname: String, index: Number = 0)
```
- objname: Must be the object's internal SMG name, for example `AstroCore` for the Comet Observatory's core.
- index: (Optional - Default: 0) If there are multiple instances of an object, such as `Coin`, use this 0-indexed incrementing value to pick between instances.

```js
function SMG_warpHere(objname, index) {
	let objectList = main.scene.sceneObjHolder.nameObjHolder.nameObjs;
	let i = index;
	if (!index) i = 0;
	let worldMatrix = main.viewer.camera.worldMatrix;
	let obj = objectList.filter((g) => g.name === objname);
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

**Usage:**
```js
SMG_allObjectsVisible() => void
```
```js
function SMG_allObjectsVisible() {
	let objectList = main.scene.sceneObjHolder.nameObjHolder.nameObjs;
	objectList.forEach(o => {
		o.visibleScenario = true; o.visibleModel = true; o.visibleAlive = true;
	});
	return 'SMG - Forced all objects to be visible.';
}
```

​
### Disable Skyboxes Following the Camera
Prevents the skyboxes from following the camera, so you can move outside of the skyboxes and zoom in on them.
> This script was provided publicly by Jasper on the noclip Discord.

> ⚠️ This script requires reloading the page to disable properly.
> 
> **Changing `false` to `true` on this script will *NOT* do what you expect!**

**Usage:** Run as shown below.
```js
main.scene.sceneObjHolder.nameObjHolder.nameObjs.forEach((v) => v.isSkybox = false)
```

​
### Change Luma Following Camera Speed
Changes the speed at which the camera follows the Lumas on the special **Day in the Life of a Luma** stage.
> This script was provided publicly by Jasper on the noclip Discord.

> ⚠️ **This script ONLY WORKS on the Day in the Life of a Luma stage.**

> ⚠️ **Setting any invalid value will freeze noclip until a page refresh.**

**Usage:** Run as shown below, replace `0.125` with the desired value.
- The default value is `0.125`
- Lower values mean slower and smoother camera following.
- Higher values mean faster and rougher camera following.
- Setting the value to `0` freezes the camera in place.
- Values higher than `1` or lower than `0` are buggy and shouldn't be used.
- Values higher than `2` or lower than `-0.1` will blackscreen until a page refresh.

```js
main.currentSceneDesc.controller.cameraK = 0.125
```

​
### Toggle all switches in use
Toggles all valid `SW_A`, `SW_B`, `SW_APPEAR` and/or `SW_DEAD` switches on the current galaxy either **on** or **off**.

**Usage:**
```js
SMG_toggleAllSwitches(act: 'on' | 'off') => void
```
- Argument MUST be either `on` or `off`, anything else will error.

```js
function SMG_toggleAllSwitches(act) {
	[
		...main.scene.sceneObjHolder.switchWatcherHolder.watchers,
		...main.scene.sceneObjHolder.nameObjHolder.nameObjs.filter(o => o.stageSwitchCtrl)
	].forEach(x => {
		const switchCtrl = x.switchCtrl ?? x.stageSwitchCtrl;
		if (switchCtrl.isValidSwitchA()) {
			switchCtrl[`${act}SwitchA`](main.scene.sceneObjHolder);
		}
		if (switchCtrl.isValidSwitchB()) {
			switchCtrl[`${act}SwitchB`](main.scene.sceneObjHolder);
		}
		if (switchCtrl.isValidSwitchAppear()) {
			switchCtrl[`${act}SwitchAppear`](main.scene.sceneObjHolder);
		}
		if (switchCtrl.isValidSwitchDead()) {
			switchCtrl[`${act}SwitchDead`](main.scene.sceneObjHolder);
		}
	});
}
```

​

​
## Paper Mario: The Thousand Year Door Scripts

### Remove background
Removes the background image of the current map.
> This script was provided publicly by Jasper on the noclip Discord.

> ⚠️ This script requires reloading the map or the tab to disable.

**Usage:** Run as shown below.
```js
main.scene.backgroundRenderer = null
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
### Change Time of Day
Changes the current time of the day. Keep in mind the information on the script above when using this one.
> This script was provided publicly by Jasper on the noclip Discord.

> ⚠️ **Setting any invalid value, including negative numbers, will freeze noclip until a page refresh.**

**Usage:** Change `240` below to any number you want. Some reference values are listed below:
- 0: Midnight (12 AM)
- 15: 1 AM **`(15 time units = 1 hour || 1 time unit = 4 minutes)`**
- 100: Sunrise (6:40 AM)
- 180: Noon (12 PM)
- 240: Initial value when loading ZWW on noclip (4 PM)
- 280: Sunset (6:40 PM)
- 360+: Midnight (Loops back to 0)

```js
main.scene.globals.g_env_light.curTime = 240
```

​
### Change Wind Power
Changes the power (strength/speed) of the wind. This affects both wind visuals but also things that react to the wind, such as flags.
> This script was provided publicly by Jasper on the noclip Discord.

> ⚠️ This script can potentially cause noclip to freeze with invalid values used, you will need to reload the page if this occurs.

Keep in mind the wind change is gradual and not instant, it **changes at approximately 100 units per second**.

For example, if you set the wind power to `1000`, it will first take **10 seconds** to actually reach that, and when reversing this (by setting wind power back to 0) it will also take **10 seconds** to actually slow back down.

**Usage:** Zero is the default value. Change `0` below to any number you want. Higher number = more powerful wind.
- Negative values behave the same as if `0` was set.
- There is no value that causes the wind to stop.
- `Infinity` works and will keep increasing the wind speed at 100 units per second constantly.

```js
main.scene.globals.g_env_light.customWindPower = 0
```

​

​
## Mario Kart Wii Scripts

### Hide the sun
Disables/enables rendering of the sun and the sun flare effect.
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** Run as shown below to hide the sun, change `false` to `true` to show it again.
```js
main.scene.baseObjects.find((v) => v.modelInstance && v.modelInstance.name.includes('sun')).modelInstance.visible = false
```

​

​
## Katamari Damacy Scripts

### Toggle Rendering Paths
Enables/disables the visual rendering of paths used by certain objects to move along.

**Keep in mind the paths will render through walls and other things, this is intended.**
> This script was provided publicly by pfedak (another noclip developer) on the noclip Discord.

> ⚠️ This is script will **not** throw an error when used on other games, but it **still won't do anything** (most likely) regardless.

**Usage:** Run as shown below to enable rendering paths, change `true` to `false` to disable it.
```js
main.viewer.scene.drawPaths = true
```

​
### Display Object Slider
Displays the object picker slider for all currently visible objects. Click **Select** to disable it.

Sliding the slider will make all objects not covered by the slider not visible until the slider is disabled.

Clicking **Select** will also make the currently selected object on the slider flash for 3 seconds and print the object in the console.
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** Run as shown below.
```js
main.debugJunk.interactiveVizSliderSelect(main.scene.objectRenderers)
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

​

​
## Pilotwings 64 Scripts

### Enable Texture Viewer
Enables the Texture Viewer menu for the current map.
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** Run as shown below.
- Run `main.ui.textureViewer.setThingList([])` to disable the Texture Viewer.

```js
main.ui.textureViewer.setThingList(main.viewer.scene.dataHolder.textureData)
```

​
### Toggle Snow Particles
Disables/enables rendering of snow particles on snowy maps.

**Usage:** Run as shown below to disable, change `false` to `true` to re-enable.
```js
main.viewer.scene.snowRenderer.visible = false
```

​
### Toggle Skybox
Disables/enables rendering of the skybox.

**Usage:** Run as shown below to disable, change `false` to `true` to re-enable.
```js
main.viewer.scene.skyRenderers.forEach(sky => sky.visible = false)
```

​
### Spawn Objects
Spawns into the map brand new objects based off their numeric ID in the game.

*I don't know of any list for these IDs, so just try some IDs and see what they are.*
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:**
```js
PW64_spawnObject(objID: Number, x: Number = 0, y: Number = 0, z: Number = 0) => String
```
- objID: Required parameter. The ID of the object to spawn. Must be between **0** and **362**.
  - The error: `Uncaught TypeError: Cannot read property 'uvmd' of undefined` means you entered an invalid ID.
- x, y and z: (Optionals - All default to `0`) The X, Y (height) and Z coordinates to spawn the object at.

```js
function PW64_spawnObject(objID, x, y, z) {
	let obj = main.viewer.scene.spawnObject(objID);
	x = x || 0; y = y || 0; z = z || 0;
	obj.modelMatrix[12] = x;
	obj.modelMatrix[14] = y;
	obj.modelMatrix[13] = z;
	return 'Sucessfully spawned object ' + objID + ' at ' + x + ' ' + y + ' ' + z
}
```

​

​
## Banjo-Tooie Scripts

### Toggle objects visibility
Enables/disables visibility of *most* objects (keeping things like main level model/skybox/etc)

**May not work on all maps.**
> This script was provided publicly by Jasper on the noclip Discord.

**Usage:** Run as shown below to hide objects, change `false` to `true` to unhide.
```js
main.scene.geoRenderers.slice(3).forEach((v) => v.visible = false)
```
