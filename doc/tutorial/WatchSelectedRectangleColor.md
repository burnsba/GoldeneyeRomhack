# Watch Selected Rectangle - cycle color over time

Just a quick note, achieving a cyclic color effect for the selected watch rectangle, in only three new lines of code. This is easy because the player struct has a timer for amount of time in the watch menu.

Project64 Demo: https://www.youtube.com/watch?v=JZ7vg6kVqqY

Everything happens in the `set_page_rectangle_colors` method.

Alpha channel isn't normally changed, so the regular rectangles will need to have the alpha explicitly set. Otherwise as you scroll through the menu it will just use the last set value.

```
// Default color for rectangle.
num_rectangles = WATCH_NUMBER_SCREENS;
vtx = (struct WatchVertex *)g_CurrentPlayer->buffer_for_watch_greenbackdrop_vertices;
for (i=0; i<(4 * num_rectangles); i++)
{
    vtx[i].color.r = 0x20;
    vtx[i].color.g = 0x70;
    vtx[i].color.b = 0x20;
    vtx[i].color.a = g_WatchBackgroundGreen;           // <- this line is new
}
```

The only other change is for the selected rectangle code. This sets the alpha based on `g_CurrentPlayer->watch_pause_time`. I'm ignoring the lower two bits, and shifting it up by two. This seems to give a smooth effect on Project64.

The remaining color code is as follows

```
i = watch_screen_index * 4;
limit = i + 3;
for ( ; i <= limit; i++)
{
    // Color of currently selected screen
    vtx[i].color.r = 0x50;
    vtx[i].color.g = 0xF0;
    vtx[i].color.b = 0x50;
    // the following line is new:
    vtx[i].color.a = (g_CurrentPlayer->watch_pause_time & 0xfc) << 2;

    if (watch_item_is_actively_selected)
    {
        // Color of currently selected screen when a menu option is selected.
        // This applies on main screen, game options, controller options, objective status, but not inventory.
        vtx[i].color.r = 0xFF;
        vtx[i].color.g = 0xFF;
        vtx[i].color.b = 0xFF;
    // the following line is new:
        vtx[i].color.a = (g_CurrentPlayer->watch_pause_time & 0xfc) << 2;
    }
}
```