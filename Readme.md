# Goldeneye Romhack and reference

Repository of Goldeneye romhack related work.

# Romhacks

Romhacks are posted as xdelta patch. You will need a suitable patch utility such as xdeltaUI to apply the patch.

Romhacks:

- [lag](lag/readme.md): Main lag testing romhack, shows mission timer, props on screen, guards on screen and more
- [foglag](foglag/readme.md): Fork of lag test romhack, used to load environment settings
- [shotvis](shotviz/readme.md): Show guard shot intolerance and audio threshold indicator
- [arlene](arlene/readme.md): r-lean practice romhack
- [valstrafe](valstrafe/readme.md): Val strafe practice. Shows Val travel distance, time of encounter, and speed.
- [benchmark](benchmark/readme.md): benchmark TAS loop for testing theories and data collection
- [splits](gsplits/readme.md): build-your-own splits practice rom
- [XLTK](XLTK/readme.md): Extended License To Kill romhack
- [OFOS Runway](ofos_runway/readme.md): Practice OFOS Runway
- [Bond speed](bondspeed/readme.md): Show Bond's actual movement speed
- [TANKBOND](tankbond/readme.md): Play the game in a tank
- [camera obj](camera_obj/readme.md): Show conditions for camera objective
- [boost](boost/readme.md): Press the DPad for a boost.

# Dev Tutorials

If you have access to decomp, these tutorials may be useful for writing your own romhack.

Tutorials:

- [Add HUD](lag/hud_readme.md): Outline of how I added hud to the lag romhack
- [Custom Watch Screen](doc/tutorial/WatchMenu.md): Walkthrough of changes required to add a custom watch screen
- [Watch screen select color](doc/tutorial/WatchSelectedRectangleColor.md): A quick note on how to make the watch screen indicator color cycle from a convenient timer
- [Disable all security cameras](doc/tutorial/DisableCctv.md): helper method to disable all cameras
- [runtime guard toggle](doc/tutorial/ToggleGuards.md): how I toggle guards at runtime
