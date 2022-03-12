# Toggle Guard Option

Here's how I implemented runtime option to enable or disable guards.

I defined a global option variable in lvl.c. Header definitions:

```
#define ROMHACK_OPTION_GUARDS          0x00000200
extern u32 g_romhack_options;
```

I couldn't prevent guards being loaded at level start, this was causing strange issues. Instead I allow guard to be setup then immediately flag the guard as dead and removed.

chrlv.c method `expand_09_characters`:

```
new_chr->headnum = (s8) headid;
new_chr->bodynum = (s8) bodyid;

if ((setup_guard->bitflags & 4) != 0)
{
    new_chr->chrflags |= CHRFLAG_CLONE;
}

if ((setup_guard->bitflags & 8) != 0)
{
    new_chr->chrflags |= CHRFLAG_INVINCIBLE;
}

// this part is new
if ((g_romhack_options & ROMHACK_OPTION_GUARDS) == 0)
{
    new_chr->actiontype = ACT_DEAD;
    new_chr->hidden |= CHRHIDDEN_REMOVE;
}
```

The next place to update is the guard constructor called from AI script, `actionblock_guard_constructor_BDBE`. I first check if this is Dr. Doak or Trevelyn on statue, then continue or exit based on the global option flag

```
s32 important_npc = 0;

if (headnum == HEAD_Male_Dave_Dr_Doak && bossGetStageNum() == LEVELID_FACILITY)
{
    important_npc = 1;
}
else if (bodynum == BODY_Trevelyan_Janus)
{
    important_npc = 1;
}

if ((g_romhack_options & ROMHACK_OPTION_GUARDS) == 0 && important_npc == 0)
{
    return NULL;
}
```

The last place I updated is the AI script to clone a guard:

```
case AI_TRYCloningChr:

    // declarations
    
if ((g_romhack_options & ROMHACK_OPTION_GUARDS) == 0)
{
    Offset += AI_TRYCloningChr_LENGTH;
    break;
}
```
