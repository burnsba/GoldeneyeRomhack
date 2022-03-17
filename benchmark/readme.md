# Benchmark ROM

**beta** release of benchmark rom.

I have written a simple "TAS" for Runway. Actions are performed after a certain number of ticks have passed.

I have removed the drones, all gaurds, all doors, and removed the AI script to spawn guards. The level forcibly aborts (same as watch quit out) when you pass a z position threshold at the end of the runway.

The level continuous loop until you press **L Trigger**. All other controller input is ignored. When you quit, the end screen will show stats: number of runs, min (best) time, max (worst) time, and average of all completed runs. The current/aborted run is not included.

# Stages

Here are the different "modes" to benchmark. Choose the level to run that test.

facility: neutral look, switch to unarmed  
runway: "default", neutral look, pp7 out  
surface: neutral look, double grenade launchers w/ firing  
surface: neutral look, double grenade launchers passive  

  
frigate: full lookdown, switch to unarmed  
surface 2: full lookdown, pp7 out  
bunker 2: full lookdown, double grenade launchers w/ firing  
statue: full lookdown, double grenade launchers passive  

# TAS

```
{
    // low level controller setup
    g_ContData[0].curstart = 0;
    g_ContData[0].curlast = 1;

    g_ContData[0].samples[1].pads[0].button = 0;
    g_ContData[0].buttonspressed[0] = 0;
    g_ContData[0].samples[1].pads[0].stick_x = 0;

    if (g_GlobalTimer < 12)
    {
        // cut cinema 1
        g_ContData[0].samples[1].pads[0].button |= A_BUTTON;
        g_ContData[0].buttonspressed[0] |= A_BUTTON;
    }
    if (g_GlobalTimer > 100 && g_GlobalTimer < 104)
    {
        // cut cinema 2
        g_ContData[0].samples[1].pads[0].button |= A_BUTTON;
        g_ContData[0].buttonspressed[0] |= A_BUTTON;
    }
    else if (g_GlobalTimer > 340 && g_GlobalTimer < 370)
    {
        g_ContData[0].samples[1].pads[0].stick_x = 25;
    }
    else if (g_GlobalTimer > 500 && g_GlobalTimer < 520)
    {
        g_ContData[0].samples[1].pads[0].stick_x = -30;
    }
    else if (g_GlobalTimer > 600 && g_GlobalTimer < 627)
    {
        g_ContData[0].samples[1].pads[0].stick_x = 50;

        // switch to unarmed only happens on certain stages
        if (switched_weapon == 0)
        {
            switched_weapon = 1;
            g_ContData[0].samples[1].pads[0].button |= A_BUTTON;
            g_ContData[0].buttonspressed[0] |= A_BUTTON;
        }
    }
    else if (g_GlobalTimer > 630)
    {
        // Left strafe in all stages.
        // CUP is only included on certain stages.
        g_ContData[0].samples[1].pads[0].button |= L_CBUTTONS | U_CBUTTONS;
        g_ContData[0].buttonspressed[0] |= L_CBUTTONS | U_CBUTTONS;

        // z trigger is only on certain stages.
        prev_action_timer++;
        if (prev_action_timer > 30)
        {
            prev_action_timer = 0;
            g_ContData[0].samples[1].pads[0].button |= Z_TRIG;
            g_ContData[0].buttonspressed[0] |= Z_TRIG;

            give_cur_player_ammo(AMMO_GRENADEROUND, get_max_ammo_for_type(AMMO_GRENADEROUND));
        }
    }

    // always hold forward
    g_ContData[0].samples[1].pads[0].stick_y = 100;
}
```