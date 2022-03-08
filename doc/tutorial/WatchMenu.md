# Custom Watch Screen Tutoral

Following is a brief explanation of the required steps to add a custom watch menu. This will cover:

- Linking forward and backward page navigation
- Navigating list of available options on a new watch screen
- Handling user input on the new watch screen (selecting an option, changing a value)
- Finishing touch: selected page marker

Only the first option will do anything in this tutorial, but this will add four rows with two values each:
```
line mode   ON    OFF
test        ON    OFF
L/R         LEFT  RIGHT
U/D         TOP   BOTTOM
```

Note: Some of the methods here are not yet fully decomp, be sure to remove #ifdef NONMATCHING.

You can see a demo of the end result on everdrive at [https://www.youtube.com/watch?v=6743cv_04H8](https://www.youtube.com/watch?v=6743cv_04H8).

# Text

I added new text at line 62 to `assets/obseg/text/LoptionsE.c`. The end of the file now looks like

```
  "1. mission objectives\n",
  0,
  "line mode\n",
  "test\n",
  "L/R\n",
  "U/D\n",
  "left\n",
  "right\n",
  "top\n",
  "bottom\n",
  0,
  0
 };
```

# Watch enums
I want to add a new page between the game options and the mission briefing. I'll start with adding an entry to `enum WATCH_INDEX`.

```
typedef enum WATCH_INDEX {
    WATCH_INDEX_MISSION_STATUS = 0,
    WATCH_INDEX_INVENTORY,
    WATCH_INDEX_CONTROL_OPTIONS,
    WATCH_INDEX_GAME_OPTIONS,
    WATCH_INDEX_ROMHACK,             // <- new value
    WATCH_INDEX_MISSION_BRIEFING
} WATCH_INDEX;
```

I also want strongly typed row definitions for the current row the user has selected on the page, so I will add a new enum to list the options rows on the page.
```
typedef enum WATCH_ROMHACK_OPTIONS_INDEX {
    ROMHACK_OPTIONS_INDEX_HUD = 0,
    ROMHACK_OPTIONS_INDEX_SPLITS,
    ROMHACK_OPTIONS_INDEX_LR,
    ROMHACK_OPTIONS_INDEX_UD,
    ROMHACK_OPTIONS_LAST
} WATCH_ROMHACK_OPTIONS_INDEX;
```

# Level init

In the `init_watch_at_start_of_stage` method I added a statement to set the default row.

```
romhack_options_index = ROMHACK_OPTIONS_INDEX_HUD;
```

The regular options are loaded from save file, but that won't be used here. If a non-default value were used, it should be setup here, but the options in this tutorial are defaulted to zero, so there's nothing else to setup.

# Watch Page Options

I setup a new `game_options` array for the new watch page. This follows how the game options page functions. There are up to four text options that will be printed on a row, and this is associated with a "current value" which is used to define what option in the row is selected. For example, "full/wide/cinema" would have current value of 0/1/2, depending on which option the user is currently selecting in the row. I also added a global variable to track which row is currently selected, called `romhack_options_index`.

```
struct game_options romhack_options_entries[] = {
    // I added text to LoptionsE starting on line 62
    { {TEXT(LOPTIONS,62), TEXT(LOPTIONS,0x1A), TEXT(LOPTIONS,0x19), 0}, 0},  // HUD
    { {TEXT(LOPTIONS,63), TEXT(LOPTIONS,0x1A), TEXT(LOPTIONS,0x19), 0}, 0},  // Splits
    { {TEXT(LOPTIONS,64), TEXT(LOPTIONS,66), TEXT(LOPTIONS,67), 0}, 0},  // L/R
    { {TEXT(LOPTIONS,65), TEXT(LOPTIONS,68), TEXT(LOPTIONS,69), 0}, 0}  // U/D
};

u32 romhack_options_index = 0;
```

# Input Handling

The user input processing starts in `sub_GAME_7F0A6A80`. There is a switch statement based on `watch_screen_index`, the currently selected watch screen. Each screen seems to handle option/row input, and then process general page input (left/right to next page). I added a new case statement:

```
case WATCH_INDEX_ROMHACK:
    switch (romhack_options_index)
    {
        case ROMHACK_OPTIONS_INDEX_LINEMODE:
        case ROMHACK_OPTIONS_INDEX_TEST:
        case ROMHACK_OPTIONS_INDEX_LR:
        case ROMHACK_OPTIONS_INDEX_UD:
            romhack_options_navigation();
    }
    watch_screen_romhack_navigation();
break;
```

The main watch graphics rendering method will set `watch_item_is_actively_selected` if the A or Z button is pressed. This is used by the watch code to check if the current row should be marked active or not.

`romhack_options_navigation` handles button presses for the page, but does not handle navigating away from the page; this follows how other watch pages are programmed. First up is checking for up and down navigation. This will always switch to the next row, and wrap around if necessary. Note that scrolling up or down clears `watch_item_is_actively_selected`.

There are many different ways to check for value changing, but I came up with a status flag that will be set when the option changes, and then resolving that back to figure out what option changed. This says, if `watch_item_is_actively_selected` is active, and the user press left, then update the user interface to change the selected value to the left. And, well if the option changed over to the left then that means the current value changed. This will set `aux=1` when the value changes like this.

At the end of the method I check if `aux == 1`. If so, I iterate over `romhack_options_index` to figure out the selected value (only 0 is implemented here). Then I know zero is linemode, so I set or clear the setting accordingly.

```
void romhack_options_navigation(void)
{
    s32 aux;

    if ((joyGetButtonsPressedThisFrame(PLAYER_1, U_CBUTTONS|U_JPAD)) || (sub_GAME_7F0A5088()))
    {
        romhack_options_index = romhack_options_index - 1;
        set_D_80040AE0_0();
        reset_watch_item_is_actively_selected();
    }
    else if ((joyGetButtonsPressedThisFrame(PLAYER_1, D_CBUTTONS|D_JPAD)) || (sub_GAME_7F0A50C4()))
    {
        romhack_options_index = romhack_options_index + 1;
        set_D_80040AE0_0();
        reset_watch_item_is_actively_selected();
    }

    aux = romhack_options_index;

    if (aux >= ROMHACK_OPTIONS_LAST)
    {
        romhack_options_index = ROMHACK_OPTIONS_INDEX_HUD;
    }
    else if (aux < 0)
    {
        romhack_options_index = (ROMHACK_OPTIONS_LAST - 1);
    }

    aux = 0;

    if ( (joyGetButtonsPressedThisFrame(PLAYER_1, L_CBUTTONS|L_TRIG|L_JPAD) || sub_GAME_7F0A4FB0()) && watch_item_is_actively_selected )
    {
        if (romhack_options_entries[romhack_options_index].current_value == 1)
        {
            game_option_select_value(&romhack_options_entries[romhack_options_index].current_value, 0);
            aux = 1;
        }
    }
    else
    {
        if ( (joyGetButtonsPressedThisFrame(PLAYER_1, R_CBUTTONS|R_TRIG|R_JPAD) || sub_GAME_7F0A4FEC()) && watch_item_is_actively_selected )
        {
            if (romhack_options_entries[romhack_options_index].current_value == 0)
            {
                game_option_select_value(&romhack_options_entries[romhack_options_index].current_value, 1);
                aux = 1;
            }
        }
    }

    if (aux)
    {
        if (romhack_options_index == ROMHACK_OPTIONS_INDEX_LINEMODE)
        {
            if (romhack_options_entries[romhack_options_index].current_value)
            {
                set_debug_VisCVG_flag(1);
            }
            else
            {
                set_debug_VisCVG_flag(0);
            }
        }
    }
}
```

The other method `watch_screen_romhack_navigation` is for navigating to a new page. The new page should have the game options on the left and mission briefing on the right, so it looks like:

```
// WATCH_INDEX_ROMHACK
void watch_screen_romhack_navigation(void) {

    if ((joyGetButtonsPressedThisFrame(PLAYER_1, L_CBUTTONS|L_TRIG|L_JPAD)) || (sub_GAME_7F0A4FB0()))
    {
        if ((joyGetButtons(PLAYER_1, Z_TRIG) == 0) && (watch_item_is_actively_selected == 0))
        {
            watch_screen_index = WATCH_INDEX_GAME_OPTIONS;
            reset_game_options_index();
            sub_GAME_7F0A5210();
            return;
        }
    }
    if ((joyGetButtonsPressedThisFrame(PLAYER_1, R_CBUTTONS|R_TRIG|R_JPAD)) || (sub_GAME_7F0A4FEC()))
    {
        if ((joyGetButtons(PLAYER_1, Z_TRIG) == 0) && (watch_item_is_actively_selected == 0))
        {
            watch_screen_index = WATCH_INDEX_MISSION_BRIEFING;
            sub_GAME_7F0A5210();
            trigger_watch_zoom(WATCHZOOM1, 15.0f);
        }
    }
}
```

The game options navigation needs to be updated so that scrolling right will end up on the new page

```
void watch_screen3_navigation(void) {

    if ((joyGetButtonsPressedThisFrame(PLAYER_1, L_CBUTTONS|L_TRIG|L_JPAD)) || (sub_GAME_7F0A4FB0()))
    {
        if ((joyGetButtons(PLAYER_1, Z_TRIG) == 0) && (watch_item_is_actively_selected == 0))
        {
            watch_screen_index = WATCH_INDEX_CONTROL_OPTIONS;
            reset_controller_options_index();
            set_controlstick_lr_disabled();
            return;
        }
    }
    if ((joyGetButtonsPressedThisFrame(PLAYER_1, R_CBUTTONS|R_TRIG|R_JPAD)) || (sub_GAME_7F0A4FEC()))
    {
        if ((joyGetButtons(PLAYER_1, Z_TRIG) == 0) && (watch_item_is_actively_selected == 0))
        {
            watch_screen_index = WATCH_INDEX_ROMHACK;
            reset_romhack_options_index();
            sub_GAME_7F0A5210();
        }
    }
}
```

And the mission briefing page needs to be updated to scroll to the left

```
void watch_screen4_navigation(void) {

    if ((joyGetButtonsPressedThisFrame(PLAYER_1, L_CBUTTONS|L_TRIG|L_JPAD)) || (sub_GAME_7F0A4FB0()))
    {
        if (watch_item_is_actively_selected == 0)
        {
            watch_screen_index = WATCH_INDEX_ROMHACK;
            reset_romhack_options_index();
            sub_GAME_7F0A5210();
            trigger_watch_zoom(WATCHZOOM3, 15.0f);
            return;
        }
    }
    if ((joyGetButtonsPressedThisFrame(PLAYER_1, R_CBUTTONS|R_TRIG|R_JPAD)) || (sub_GAME_7F0A4FEC()))
    {
        if (watch_item_is_actively_selected == 0)
        {
            watch_screen_index = WATCH_INDEX_MISSION_STATUS;
            zero_D_800409A4();
            sub_GAME_7F0A5210();
            trigger_watch_zoom(WATCHZOOM2, 15.0f);
        }
    }
}
```

The `reset_romhack_options_index` method is just
```
void reset_romhack_options_index(void) {
    romhack_options_index = 0;
}
```

# Graphics Rendering

The main watch graphics method is `sub_GAME_7F0ACA28`. There is a switch statement to render the watch screen based on `watch_screen_index`. Added a new case statement for the new screen

```
switch (watch_screen_index)
{
    case WATCH_INDEX_MISSION_STATUS:
        gdl = draw_watch_mission_status_page(gdl, arg1);
        break;
    case WATCH_INDEX_INVENTORY:
        gdl = draw_watch_inventory_page(gdl, arg1);
        break;
    case WATCH_INDEX_CONTROL_OPTIONS:
        gdl = draw_watch_control_options_page(gdl, arg1);
        break;
    case WATCH_INDEX_GAME_OPTIONS:
        gdl = draw_watch_game_options_page(gdl, arg1);
        break;
    case WATCH_INDEX_ROMHACK:
        gdl = draw_watch_romhack_options_page(gdl, arg1);
        break;
    case WATCH_INDEX_MISSION_BRIEFING:
        gdl = draw_watch_mission_briefing_page(gdl, arg1);
}
```

I tried to keep the graphics rendering simple. This is easily accomplished because all the new option rows follow the same format. They have a text label, and then exactly two text values. The `current_value` property can be used to tell which item in the row is selected. The `jp_text_write_stuff` method is called when the row is active and the value in the row is selected (off, or on); this adds a highlight around the text. Otherwise `en_text_write_stuff` is called.

Rows are spaced vertically by 16. Values within the row are spaced by 60.

The entire method looks like

```
Gfx *draw_watch_romhack_options_page(Gfx *gdl, s32 param_2)
{
    gdl = draw_background_health_and_armor(gdl, param_2, 0);

    if (check_watch_page_transistion_running() != 1)
    {
        s32 sp5C;
        u8 *textptr;
        s32 draw_x;
        s32 draw_y;
        s32 sp4C;
        s32 sp48;
        s32 option_value;
        s32 option_index;

        gdl = microcode_constructor(gdl);

        for (option_index = 0; option_index < ROMHACK_OPTIONS_LAST; option_index++)
        {
            textptr = langGet(romhack_options_entries[option_index].text[0]); // title

            draw_x = XOFFSET_1;
            draw_y = YOFFSET_8 + (option_index * 16);

            sp5C = 0xFF00B0;

            if (romhack_options_index == option_index)
            {
                sp5C = 0xA0FFA0F0;
                if (watch_item_is_actively_selected)
                {
                    sp5C = -1;
                }
            }

            sub_GAME_7F0AE98C(&sp48, &sp4C, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, 0);

            if ((watch_item_is_actively_selected) && (romhack_options_index == option_index))
            {
                gdl = jp_text_write_stuff(gdl, &draw_x, &draw_y, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, sp5C, 0x7000A0, sp4C + 1, sp48, 0, 0);
            }
            else
            {
                gdl = en_text_write_stuff(gdl, &draw_x, &draw_y, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, sp5C, sp4C, sp48, 0, 0);
            }

            ////////////////////////////////////

            option_value = romhack_options_entries[option_index].current_value == 0;

            draw_x = XOFFSET_1 + 60;
            draw_y = YOFFSET_8 + (option_index * 16);

            textptr = langGet(romhack_options_entries[option_index].text[1]); // option 1

            sp5C = option_value ? 0xA0FFA0F0 : 0x60A060B0;

            sub_GAME_7F0AE98C(&sp48, &sp4C, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, 0);

            if ((watch_item_is_actively_selected) && (romhack_options_index == option_index) && option_value)
            {
                gdl = jp_text_write_stuff(gdl, &draw_x, &draw_y, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, sp5C, 0x7000A0, sp4C + 1, sp48, 0, 0);
            }
            else
            {
                gdl = en_text_write_stuff(gdl, &draw_x, &draw_y, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, sp5C, sp4C, sp48, 0, 0);
            }

            ////////////////////////////////////

            option_value = romhack_options_entries[option_index].current_value == 1;

            draw_x = XOFFSET_1 + 120;
            draw_y = YOFFSET_8 + (option_index * 16);

            textptr = langGet(romhack_options_entries[option_index].text[2]); // option 2

            sp5C = option_value ? 0xA0FFA0F0 : 0x60A060B0;

            sub_GAME_7F0AE98C(&sp48, &sp4C, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, 0);

            if ((watch_item_is_actively_selected) && (romhack_options_index == option_index) && option_value)
            {
                gdl = jp_text_write_stuff(gdl, &draw_x, &draw_y, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, sp5C, 0x7000A0, sp4C + 1, sp48, 0, 0);
            }
            else
            {
                gdl = en_text_write_stuff(gdl, &draw_x, &draw_y, textptr, ptrSecondFontTableSmall, ptrFirstFontTableSmall, sp5C, sp4C, sp48, 0, 0);
            }
        }
    }

    return gdl;
}
```

# Finishing touches

I fixed an issue with line mode which causes the watch menu to be unusable. In bossMainLoop I changed the line mode check from
```
if (get_debug_VisCVG_flag())
```
to
```
if (g_CurrentPlayer->outside_watch_menu == 1 && get_debug_VisCVG_flag())
```

-----

The watch uses a buffer in the player struct for storing verteces and DL list for several watch items. Primarily this is the five page select rectangles at the bottom of the screen, but there is a separate section for static, the music option, and sound effect option.

Because the player struct is not fully shiftable, I had to continue the screen select rectangles elsewhere. I added these to .data in bondview.c. Note: I was having a graphics issue; I think this might be due to alignment, but not using index zero of the new vertex array seems to fix the issue. No idea.

(#defines explained below)  

```
// +1 for end display list command
Gfx g_romhack_extract_buffer_for_watch_greenbackdrop_DL[WATCH_ROMHACK_NUMBER_SCREENS + 1] = {0};
// +1 because I can't figure out why starting at index zero causes graphics glitch
struct WatchRectangle g_romhack_buffer_for_watch_greenbackdrop_vertices[WATCH_ROMHACK_NUMBER_SCREENS + 1] = {0};
```

The watch.h file needs to be updated to add new #define definitions since the `WATCH_NUMBER_SCREENS` can't be modified. A new value is added to count the number of new screens being added, and the rectangle and spacer macros were updated. The header should now have

```
#define WATCH_NUMBER_SCREENS 5

#define WATCH_ROMHACK_NUMBER_SCREENS 1

// default: 100
#define WATCH_SCREEN_SELECT_RECTANGLE_WIDTH   (s32)(WATCH_SCREEN_SELECT_TOTAL_WIDTH / (WATCH_NUMBER_SCREENS + WATCH_ROMHACK_NUMBER_SCREENS + 1))
// default: 25
#define WATCH_SCREEN_SELECT_SPACER_WIDTH      (s32)(WATCH_SCREEN_SELECT_RECTANGLE_WIDTH / (WATCH_NUMBER_SCREENS + WATCH_ROMHACK_NUMBER_SCREENS - 1))
```

Add the extern definitions for the above to bondview.h for use in watch.c.

I also updated `trigger_solo_watch_menu` method in bondview, which sets up the rectangles for rendering on screen. I changed it to append the new rectangle after the current five rectangles when spacing them horizontally across the screen. The entire case 0 of watch animation now looks like

```
if (g_CurrentPlayer->watch_animation_state == 0)
    {
        if (arg0 == 0)
        {
            s32 rectangle_offset;

            watch_transition_time *= 1.1f;
            if (watch_transition_time > 1.7f)
            {
                watch_transition_time = 1.7f;
            }

            if ((Gun_hand_without_item(1) != 0)
                && (Gun_hand_without_item(0) != 0)
                && (g_CurrentPlayer->hands[1].when_detonating_mines_is_0 != 5)
                && (g_CurrentPlayer->hands[1].when_detonating_mines_is_0 != 6)
                && (g_CurrentPlayer->hands[1].when_detonating_mines_is_0 != 7)
                && (g_CurrentPlayer->hands[1].when_detonating_mines_is_0 != 8))
            {
                g_CurrentPlayer->watch_animation_state = 1;
            }
            else
            {
                g_CurrentPlayer->watch_animation_state = 0xD;
            }

            g_CurrentPlayer->watch_pause_time = 0;
            g_CurrentPlayer->field_1C4 = 0;

            sub_GAME_7F07DEFC();
            bondviewTriggerWatchZoomDefault();

            sub_GAME_7F0A2F30(&g_CurrentPlayer->healthdamagetype, 0x2E, 1, get_BONDdata_watch_armor());
            sub_GAME_7F0A3330(&g_CurrentPlayer->watch_body_armor_bar_gdl, OS_K0_TO_PHYSICAL(&g_CurrentPlayer->healthdamagetype), 0x2E);

            sub_GAME_7F0A2F30(&g_CurrentPlayer->related_to_health_display, 0x2E, -1, bondviewGetCurrentPlayerHealth());
            sub_GAME_7F0A3330(&g_CurrentPlayer->watch_health_bar_gdl, OS_K0_TO_PHYSICAL(&g_CurrentPlayer->related_to_health_display), 0x2E);

            sub_GAME_7F0A69A8();

/**
 * Horizontal spacing between watch menu screen select rectangles.
 * Default = 125.
*/
#define WATCH_SCREEN_SELECT_RECTANGLE_HSTEP (WATCH_SCREEN_SELECT_RECTANGLE_WIDTH + WATCH_SCREEN_SELECT_SPACER_WIDTH)

            /**
             * This section is for rendering the selected screen rectangles.
            */
            ptr_b = g_CurrentPlayer->buffer_for_watch_greenbackdrop_DL;
            ptr_a = g_CurrentPlayer->buffer_for_watch_greenbackdrop_vertices;

            rectangle_offset = 0;

            for (i=0;
                i<WATCH_NUMBER_SCREENS;
                i++,
                    ptr_a = next)
            {
                ptr_copy = ptr_a;
                // Note: colors are set here but overwritten in watch.c set_page_rectangle_colors
                next = setup_watch_rectangles(ptr_a, rectangle_offset, 0, WATCH_SCREEN_SELECT_RECTANGLE_WIDTH, 0x14, WATCH_SCREEN_SELECT_RECTANGLE_MIN_X, 0x136);
                ptr_b = sub_GAME_7F0A3B40(ptr_b, OS_K0_TO_PHYSICAL(ptr_copy));
                rectangle_offset += WATCH_SCREEN_SELECT_RECTANGLE_HSTEP;
            }

            gSPEndDisplayList(ptr_b);
            /**
             * End watch screen select rectangles.
            */

            /**
             * Continue rendering extra romhack watch screen rectangles.
            */
            ptr_b = g_romhack_extract_buffer_for_watch_greenbackdrop_DL;
            // +1 because I can't figure out why starting at index zero causes graphics glitch
            ptr_a = &g_romhack_buffer_for_watch_greenbackdrop_vertices[1];

            for (i=0;
                i<WATCH_ROMHACK_NUMBER_SCREENS;
                i++,
                    ptr_a = next)
            {
                ptr_copy = ptr_a;
                // Note: colors are set here but overwritten in watch.c set_page_rectangle_colors
                next = setup_watch_rectangles(ptr_a, rectangle_offset, 0, WATCH_SCREEN_SELECT_RECTANGLE_WIDTH, 0x14, WATCH_SCREEN_SELECT_RECTANGLE_MIN_X, 0x136);
                ptr_b = sub_GAME_7F0A3B40(ptr_b, OS_K0_TO_PHYSICAL(ptr_copy));
                rectangle_offset += WATCH_SCREEN_SELECT_RECTANGLE_HSTEP;
            }

            gSPEndDisplayList(ptr_b);
            /**
             * End romhack watch screen select rectangles.
            */

            /**
             * This section is related to rendering static on the watch menu.
             * Static is defined by a horizontal bar in the middle of the screen.
            */
            ptr_a = g_CurrentPlayer->buffer_for_watch_static_vertices;
            ptr_b = g_CurrentPlayer->buffer_for_watch_static_DL;

            ptr_copy = g_CurrentPlayer->buffer_for_watch_static_vertices;
            next = setup_watch_rectangles(ptr_a, 0, 0, 0x398, 0x14, -0x1CC, 0);
            ptr_b = sub_GAME_7F0A3B40(ptr_b, OS_K0_TO_PHYSICAL(ptr_copy));

            gSPEndDisplayList(ptr_b);
            /**
             * End watch static section.
            */
        }
    }
```

I ended up changing the signature for `set_page_rectangle_colors`. Previously this accepted a pointer to an array of verteces, but I change it to only accept the screen index. The method called in `sub_GAME_7F0ACA28` is therefore changed to `set_page_rectangle_colors(watch_screen_index);`.

The `set_page_rectangle_colors` method is used to set colors for the rectangle at the bottom of the page. When the current page is selected it will be brighter. Unless an option is actively selected, then it will be darker. I had to modify the method to iterate over the previous player struct array, but then continue iteration in the new vertex array setup in bondview. The method now looks like

```
void set_page_rectangle_colors(s32 watch_screen_index)
{
    s32 i;
    s32 limit;
    s32 num_rectangles;
    struct WatchVertex *vtx;

    // Default color for rectangle.
    num_rectangles = WATCH_NUMBER_SCREENS;
    vtx = (struct WatchVertex *)g_CurrentPlayer->buffer_for_watch_greenbackdrop_vertices;
    for (i=0; i<(4 * num_rectangles); i++)
    {
        vtx[i].color.r = 0x20;
        vtx[i].color.g = 0x70;
        vtx[i].color.b = 0x20;
    }

    num_rectangles = WATCH_ROMHACK_NUMBER_SCREENS;
    // +1 because I can't figure out why starting at index zero causes graphics glitch
    vtx = (struct WatchVertex *)&g_romhack_buffer_for_watch_greenbackdrop_vertices[1];
    for (i=0; i<(4 * num_rectangles); i++)
    {
        vtx[i].color.r = 0x20;
        vtx[i].color.g = 0x70;
        vtx[i].color.b = 0x20;
    }

    vtx = NULL;

    if (watch_screen_index >= 0 && watch_screen_index < WATCH_NUMBER_SCREENS)
    {
        vtx = (struct WatchVertex *)g_CurrentPlayer->buffer_for_watch_greenbackdrop_vertices;
    }
    else
    {
        watch_screen_index -= WATCH_NUMBER_SCREENS;

        if (watch_screen_index >= 0 && watch_screen_index < WATCH_ROMHACK_NUMBER_SCREENS)
        {
            // +1 because I can't figure out why starting at index zero causes graphics glitch
            vtx = (struct WatchVertex *)&g_romhack_buffer_for_watch_greenbackdrop_vertices[1];
        }
    }

    if (vtx == NULL)
    {
        return;
    }

    i = watch_screen_index * 4;
    limit = i + 3;
    for ( ; i <= limit; i++)
    {
        // Color of currently selected screen
        vtx[i].color.r = 0x50;
        vtx[i].color.g = 0xF0;
        vtx[i].color.b = 0x50;

        if (watch_item_is_actively_selected)
        {
            // Color of currently selected screen when a menu option is selected.
            // This applies on main screen, game options, controller options, objective status, but not inventory.
            vtx[i].color.r = 0x30;
            vtx[i].color.g = 0xA0;
            vtx[i].color.b = 0x30;
        }
    }
}
```

The last change is to add the new rectangles to the display list. This is handled in `draw_background_health_and_armor`. After the player struct rectangles are added, I add a reference to the new rectangle list. The relevant section now looks like

```
/**
* This section renders the green rectangles/page select at the bottom of the screen.
* This is setup in bondview trigger_solo_watch_menu.
*/
gDPSetRenderMode(gdl++, G_RM_XLU_SURF, G_RM_XLU_SURF2);
gDPSetCombineMode(gdl++, G_CC_SHADE, G_CC_SHADE);
gSPDisplayList(gdl++, OS_PHYSICAL_TO_K0(g_CurrentPlayer->buffer_for_watch_greenbackdrop_DL));
gSPDisplayList(gdl++, OS_PHYSICAL_TO_K0(g_romhack_extract_buffer_for_watch_greenbackdrop_DL));
/**
* // end green rectangles/page select section
*/
```

# Done

That's all the changes. See the youtube link at the start of the tutorial for a demo on console with  Everdrive64.
