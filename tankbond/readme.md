# TANKBOND romhack

This is a quick romhack to play the game in a tank.

Youtube demo: https://www.youtube.com/watch?v=qbeC5Tavnq0

source: https://tolos.me/goldeneye_romhack/tankbond/index.html

# Version

The NTSC-U xdelta patch should be applied against US NTSC .z64 with SHA1 `ABE01E4AEB033B6C0836819F549C791B26CFDE83`.

# Info

You enter the tank at the start of the level and can't exit the tank. You are given tank ammo on every stage except Archives and Bunker 2. You can pick up items and ammo like a normal play through.

The tank is scaled down quite a bit. Clipping code has been modified, the tank bounding box is smaller than the model. The stan navigation code has also been modified to use a hybrid between the tank code and bond code. You can drive up ramps/stairs and warp up and down ladders. 

Minimum lower angle in the tank has been lowered to -40 degrees from -20. You can rotate the tank slightly faster now.

Version 1.1

- Require expansion pak; fix texture issues (thanks Axdoomer)
- Dont collide with characters if not moving (fixes scientists)
- Adjust tiny Bond perspective, looks closer to riding regular tank now, but smaller.
- Adjust tank max speed
- Improved stan navigation (fixes sticky area in Silo, under shuttle in Aztec, exiting Depot warehouse ...)
- Ignore guard armor on tank collision (fixes Xenia, Jaws, ...)
- Disable tank collisions with Boris on Bunker 1.
- Disable tank collisions with Natalya on Bunker 2, Archives, Train, Jungle, Control.
- Collisions with doors no longer stop them from moving (technically, tank collisions disabled except during movement check)
- Enable tank collisions with Jungle drone guns.
- Enable collisions with jungle ammo dump boxes.
- Disable collisions with Silo satellite.
- Silo circuit boards are now invincible.

Version 1.0

- There are a few places in the game that are not handled very well, but every stage is completable. Known challenge areas: Aztec under the shuttle, Depot exiting the safe room into the warehouse, and you can no longer warp back to the top of the pit at the start of Aztec.
- There is not enough texture memory on some levels, so there will be noticeable artifacts. This affects the on screen crosshair and weapon ammo icon on screen. You can see other errors, such as bullet tracers and the cloud texture on Dam. I tried a few different things to fix this without success. I wanted to keep this romhack short in development time, so just going to release with this issue. Levels affected: Dam, Surface 1/2, Archives, Depot, Jungle.
- Max tank speed hasn't been changed

# Screenshots

![dam](tankbond-dam.jpg)

![dam](tankbond-facility.jpg)

![dam](tankbond-jungle.jpg)

![dam](tankbond-train.jpg)

![dam](tankbond-cradle.jpg)

![dam](tankbond-aztec.jpg)
