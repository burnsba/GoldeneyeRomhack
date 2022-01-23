Quick note on how I add the HUD.

I added a method to `fr.c` to draw the background. Note that `gDPSetRenderMode` with `NOOP` is required to work on hardware. The parameters to `GPACK_RGBA5551` set the background color used.

```
    Gfx *viDrawHudBackground(Gfx *gdl)
    {
        s32 y = viGetViewTop();
        s32 width = viGetX() - 3;

        gDPPipeSync(gdl++);
        /**
         * Manual says:
         * For fill mode and copy mode, please use g*DPSetRenderMode (G_RM_NOOP,G_RM_NOOP2). If you try to use the Z buffer with fill mode, the RDP pipeline might hang. 
        */
        gDPSetRenderMode(gdl++, G_RM_NOOP, G_RM_NOOP2);
        gDPSetCycleType(gdl++, G_CYC_FILL);
        gDPSetFillColor(gdl++, ((GPACK_RGBA5551(210, 210, 210, 1) << 16) | GPACK_RGBA5551(210, 210, 210, 1)));
        gDPFillRectangle(
            gdl++,
            1,
            y,
            width,
            y+10);
        gDPPipeSync(gdl++);

        return gdl;
    }
```

In `lvl.c` I added a global variable

```
    s32 g_RomHack_ShowHud = 0;
```

In `lvlStageLoad` I check if this is the title stage and enable/disable the flag:

```
    if (stage == LEVELID_TITLE)
    {
        g_RomHack_ShowHud = 0;

        init_menus_or_reset();
    }
    else
    {
        if (getPlayerCount() == 1)
        {
            g_RomHack_ShowHud = 1;
        }
    ...
```

In `lvlUnloadStageTextData` I clear the HUD flag:

```
    void lvlUnloadStageTextData(void)
    {
        g_RomHack_ShowHud = 0;
        ...
```

In the boss main loop I check for the flag. This code is inserted *after* the level is rendered, and *before* passing the gfx list to the RSP task for processing. Here's an example of how to write a text string to the HUD area. X and y coordinates are specified relative to the current screen view; text buffer used is `char hud_text[100]` declared at start of boss main.

Calling `microcode_constructor` is necessary before rendering text.

arg6 (`0xff`) is 32 bit rgba color of text, here r=0, g=0, b=0 since 0xff is 0x000000ff.

```
    gdl = lvlRender(gdl);

    if (g_RomHack_ShowHud)
    {
        s32 x = 4;
        s32 y = viGetViewTop();
        s32 width;
        s32 height;
        s32 vi_get_x = viGetX();

        // draw HUD background rectangle
        gdl = viDrawHudBackground(gdl);
        
        // set render mode for writing text to screen
        gdl = microcode_constructor(gdl);

        height = y + 14; // same for all.

        // Write foward/back speed
        sprintf(hud_text, "^ %04.2f %c", g_CurrentPlayer->speedforwards, '\0');

        width = vi_get_x - x - 2;
        gdl = en_text_write_stuff(gdl, &x, &y, hud_text, ptrSecondFontTableSmall, ptrFirstFontTableSmall, 0xff, width, height, 0, 0);
        
        ...
```