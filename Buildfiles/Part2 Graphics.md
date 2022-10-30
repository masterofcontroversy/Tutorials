# Graphics

## 4bpp Images
When you run `Png2Dmp` on an image with no extra arguments, a 4bpp formatted file of the image is made. This can be inserted in your hack.  
The two ways of installing these graphics are:  

Use `#incext`
```
ALIGN 4
MyImage:
#incext Png2Dmp "MyImage.png"
```

or run `Png2Dmp` on the file in advance then `#incbin` it
```
ALIGN 4
MyImage:
#incbin "MyImage.dmp"
```

A very small amount graphics are inserted uncompressed in the 4bpp format.

## lz77 Images
lz77 compressed image dump files are made when you run `Png2Dmp` with the `--lz77` arugment. They can be installed the same way as 4bpp formatted files.

Using `#incext` is slightly different than last time, so here's an example:  
```
ALIGN 4
MyImage:
#incext Png2Dmp --lz77 "MyImage.png"
```

Most graphics you can insert need to be lz77 compressed.


## Palettes
`Png2Dmp` can also output palettes of images using the `--palette-only` argument. When using `Png2Dmp` from a console to output a palette file, you must use the `-po` argument.
```
Png2Dmp MyImage.png -po <output file name>
```

You can `#incext` palettes like so:
```
ALIGN 4
MyPalette:
#incext Png2Dmp --palette-only "MyImage.png"
```

## Item Icons
<!--* Resources for the individual image method:

<!--* #ifndef IconTable
<!--*     #define IconTable $5926F4
<!--* #endif

<!--* #ifndef SetIcon
<!--*     #define SetIcon(ID) "ORG IconTable + ID * 128"
<!--* #endif

<!--* Keep in mind this only works with each icon having their own image-->

Icons are dumped in the 4bpp format, so you shouldn't use `--lz77` to compress them.  
There are several ways of inserting item icons, so I'll only be covering two.  

The first method is to use this macro to `ORG` to the item icon table:
```
#define SetIcon(ID) "ORG IconTable + ID * 128"
```

By default, the icon table is located at `0x5926F4`. If you want, you can repoint to a different address to have 255 icons total.

The next step is to insert the icon directly after. In practice, this would look like:
```
#define SetIcon(ID) "ORG IconTable + ID * 128"

PUSH //PUSHing because SetIcon ORGs
SetIcon(IronSwordIcon)
#incbin "Images/Swords/IronSword.4bpp"
POP
```

`ID` is the ID of the item Icon you wish to insert. Unlike most other graphics, the icon table does not contain pointers to graphics, but contains the graphics themselves.  
This makes installation quite simple.

This method is very useful if you have a setup where you can dump an entire group of images automatically (with a script or makefile for example).

The second method is using icon sheets. Since the icon table is a bunch of icons in a row, you can divide the sheet as many ways as you want.  
You could do
```
PUSH
ORG IconTable
#incbin "IconSheet.dmp"
POP
```
or even
```
PUSH
ORG IconTable
#incbin "IconSheet1.dmp"

#incbin "IconSheet2.dmp"

#incbin "IconSheet3.dmp"
POP
```
Naturally, you can also use `#incext` for the same result


## Portraits
For inserting portraits, we have tool named `portrait-formatter` that takes a formatted portrait (that looks like [this](https://cdn.discordapp.com/attachments/206588291053649921/821522666720460810/Eirika.png)), and formats it for GBAFE to use.  

There's also a macro named `setMugEntry` that is used to add portraits to the portrait table.
```
setMugEntry(<mugindex>,<portrait label>,<mouth X>,<mouth Y>,<eye X>,<eye Y>)
```

-	The first argument is `<mugindex>`. This is the ID the portrait will use in the portrait table.  
-	`<portrait label>` is the label where portrait data starts, meaning that what's under it (the mug itself) will be used.  
-	The next two arguments `<mouth X>,<mouth Y>` are the coordinates of the mouth. The game needs this information for the mouth animations.  
-	The last two arguments `<eye X>,<eye Y>` are the same thing as the mouth, but for the eyes.  

Insertion of portraits using `#incext` looks like:
```
ALIGN 4
MyPortrait:
#incext portrait-formatter "MyPortrait.png"

setMugEntry(0x2,MyPortrait,2,5,2,3) //Example use of setMugEntry
```

If you run `portrait-formatter` outside of `#incext`, you'll notice that there are four .dmp files that get created.  
These must be `#incbin`'d in a particular order. (Protip: If you use the `-o` flag to name an output file, all the .dmp files will be merged together and you can just `#incbin` that).

Insertion of pre-dumped portraits looks like:
```
ALIGN 4
MyPortrait:
#incbin "My_mug.dmp"
#incbin "My_frames.dmp"
#incbin "My_palette.dmp"
#incbin "My_minimug.dmp"

setMugEntry(0x2,MyPortrait,2,5,2,3) //Example use of setMugEntry
```
Remember: The order is important.

## Class Cards

Class cards are handled very similarly to portraits.  

To be inserted, class cards must be lz77 compressed, and you'll need to include the palette seperately.  

To make adding class cards to the portrait table easier, @Snaky1 made an easy to use macro:
```
#define setCardEntry(cardEntry,cardLocation,cardPaletteLocation) "PUSH; ORG PortraitTable+cardEntry*0x1C; POIN 0 0 cardPaletteLocation 0 cardLocation; POP"
```

-	`cardEntry` is the ID the class card will use in the portrait table.  
-	`cardLocation` is the label of your class card's graphic.  
-	`cardPaletteLocation` is the label of your class card's palette.

Installation of a class card using `#incext` looks like:
```
ALIGN 4
ClassCardGraphic:
#incext --lz77 "ClassCard.png"

ALIGN 4
ClassCardPalette:
#incext --palette-only "ClassCard.png"

setCardEntry(0x78,ClassCardGraphic,ClassCardPalette)
```

Installation of an already processed class card looks like:
```
ALIGN 4
ClassCardGraphic:
#incbin "ClassCard.dmp"

ALIGN 4
ClassCardPalette:
#incbin "ClassCardPalette.dmp"

setCardEntry(0x78,ClassCardGraphic,ClassCardPalette)
```

Remember that you need to run Png2Dmp on the same .png file twice with different arguments each time. (`--lz77` and `-po`).

## Map Sprites

To insert map sprites, first you need to have a properly formatted map sprite sheet. You can find plenty of them in the graphics repository.  
Keep in mind there are two sheets you'll need: One for standing map sprites, and one for moving map sprites.  

Next, you'll need to put them somewhere and compress them properly. In this case, with lz77 compression using png2dmp. Like so:
```
ALIGN 4
StandingSprite:
#incext png2dmp --lz77 "StandingSprite.png"
```

You'll need to do the same for the moving map sprite.

Now that the map sprites are inserted, we have to put them to use. I'll be using the Easy Buildfile to demonstrate.  
The Easy Buildfile offers two ways to use map sprites. First, I'll cover the CSV method.  

<!-- Keep in mind Vesly removed the functionality of the map sprite CSV table -->

Go to `Tables/Map Sprite`. There, you'll find some CSV tables for map sprites. For now, open `Standing map sprite editor.csv`.  
Don't mind the `Unknown` columns, we're only concerned about `Size` and `Pointer to graphics`.  

Pointer to graphics is self-explanitory. You type in your label name as a pointer (`LabelName|IsPointer`).

Size is a bit different. There are three options for size:
```
Size0 = 16x16 (16x48 images)
Size1 = 16x32 (16x96 images)
Size2 = 32x32 (32x96 images)
```

Pick one of these options based on the size of your standing map sprite.

Next we have `Misc map sprite editor.csv`.  
In the `Animation Pointer` column, point to your moving map sprite.

As for the other pointer (Or `Another Pointer`), that will be explained soon.

On to the second method courtesy of @Vesly.  
Go to `Graphics/MapSprites`, and open at `Readme.md`. It contains the instructions to use the map sprite tools in the folder.  

Remember how I mentioned the `Another Pointer`? Well what it does is decide where each frame of the standing map sprite is going to be (among other things)  
There isn't much of a guideline for them, and that's beyond the scope of this tutorial anyway. What I will show you is a definition for each AP (Another Pointer) used in FE8U:  

```
//Vanilla FE8 AP definitions
#define NormalAP        $81C3D7C
#define GeneralAP       $81C8A80
#define RangerAP        $81CA124
#define ArcherAP        $81D0A7C
#define M_SageAP        $81D1D48
#define FemaleSniperAP  $81D2714
#define WyvernRiderAP   $81D403C
#define WyvernLordAP    $81D5E58
#define WyvernKnightAP  $81D7CC8
#define MageAP          $81D8668
#define F_SageAP        $81D8668
#define GreatLord       $81C52B4
#define JournymanAP     $81E173C
#define DancerAP        $81ED1C8
#define BardAP          $81E8840
#define CyclopsAP       $81F49B8
#define DemonKingAP     $81FD028
```

It should be noted that you can migrate these map sprite setups to your own buildfile if you want.

## Battle Animations
This section is a step-by-step section on inserting battle animations.  

### What you'll need

First, we'll need [Animation Assembler](https://feuniverse.us/t/fe7-8-animation-assembler-convert-feditor-format-for-insertion-with-ea/1880).

Next, you'll need the animation you want to insert. You can find plenty in the [Fire Emblem Resource Repository](https://github.com/Klokinator/FE-Repo).  
The animation folders hold a lot, so I'll quickly explain what they contain:  

-	Animation frames: Combat animations are done in a frame-by-frame style. These are the frames that are used for that animation.  
-	Animation sheets: Same as the animation frames, but in a format that when compressed, the game can use.  
-	Animation script: These are used to tell the game what frames of animation to use when, and where. There's a human readable version in a .txt file, and there's a .bin version. We need to use the .bin version.  

If the animation you want to install doesn't have have the .bin file or sheets, but still has the .txt script file and the animation frames, you can import the animation to FEBuilderGBA, then export as bin. This should output the files we need.  

You'll be wanting to include the `Master Animation Installer.event` file somewhere in your buildfile as well.  

### Inserting the animation

Drag and drop the animation's .bin file onto AA.exe (Or run it in the command prompt if you prefer), and open the generated Installer.event file. Near the top you'll find `AnimTableEntry(0x0) //CHANGE THIS TO THE SLOT YOU ARE REPLACING`. I think this speaks for itself. (SLOT = Animation ID)  

After that, include the generated Installer.event in your buildfile.  

It's worth noting that quite often the animation .bin files will be named after the weapon type. This means there will likely be a lot of labels with the same name, and that's a no-no.  

One solution is to rename all the labels so they'll be unique, and another is to include the Installer.event files with curly braces `{}` around each of them to make sure the labels aren't global.  

### Battle Animation pointers

Battle animation pointers tell the game what animations to use for what items/weapons. They follow this format:

```
Animation ID, Weapon Type/Item ID, Type Designation
```

-`Animation ID` is the animation ID you want.

-`Weapon Type/Item ID` is used for one of two things. The first is the weapon type being used, and the second is a specific item.  

-`Type Designation` decides whether the last option will be used for weapon type (`Weapon Type`) or a specific item (`Item ID`)  
The options are:
```
1 = all items of that type.
0 = Specific item.
```

The terminator to end the animation pointer list is word's worth of 0 (eg. `0x00000000`). Remember to label your animation pointers (and `ALIGN 4` them too)  

Here's an example of Ephraim lord's animation pointer:
```
AddClassAnimation(AnimationID,WeaponType,TypeDesignation)
EphriamLordAnimPointer:
AddClassAnimation(EphraimLord_Lance,Lance,1) //Using 1 for all lances
AddClassAnimation(EphraimLord_Unarmed,Unarmed,1)
EndClassAnimation
```

EAstdlib has a host of animation pointer macros you can use in `EventAssembler/EAStandardLibrary/AnimationSetters.txt`

To have the game use your new animation pointer, it must be assigned to a class in the class table. This is easily done in the ClassEditor CSV table. Go to the `Animation Pointer` column (or `Battle Anims` for skillsys), find the right row for your class, and type the name of your animation pointer label. (Remember to put `|IsPointer` at the end)

You can also org to the class table and point to your animation pointer like so:
```
PUSH

ORG ClassTable + (ID * 84) + 52 //bytes 0x34 through 0x37 is where the animation pointer is in the class struct
POIN AnimPointer

POP
```

There's actually a macro for this in `AnimationSetters.txt`:
```
#define SetClassAnimation(ClassID,Pointer) "PUSH; ORG ClassTable+(ClassID*84)+52; POIN Pointer; POP"
```

This is basically what editing the table does, but more laser-focused.  
NOTE: if you're using the ClassEditor CSV, this must be done after the table is inserted, or else it will be overwritten  

### Battle Animation Palettes

Everyone having the same color palette is boring, so let's fix that.

Thanks to compression, every battle animation palette is actually five palettes in one (Though only the first four seem to be used).  
The palettes are pointed in a word from bytes 0xC to 0x10 of each entry (The first 12 bytes are for text so humans can identify the palettes)  

NOTE: The following applies to FE8 only:  
There are two other important tables. One for palette IDs to use, and the other for what classes will use those palette IDs.  
It's easier to get an idea of this by looking at the palette association tables.

### Making and Inserting Palettes

First, you'll need a palette string. It should look something like this:
```
//Colm palette
5553FF7FFF6B7C36E728AD76C9614541567B6F5E88412541C95973364B15A514
```

This is pretty easy to get via FE Recolor, FEBuilder, or...FEditor.

To insert these palettes, we have TeraSpark's [pal2EA](https://feuniverse.us/t/pal2ea-the-buildfile-palette-inserter/2646) tool.

Now let's do a quick rundown of how this works. Let's make a new file and name it something like `Palettes.event`  

Here, we can type something similar to the following:
```
#char{0x3D} "Colm_Thief" set{Colm, 0x1, Thief}
	5553FF7FFF6B7C36E728AD76C9614541567B6F5E88412541C95973364B15A514
```

We start off our palette entry with a `#`. Then we use `char` to decide what ID the palette will have. In this case, it's `0x3D`  
In between the quotes we're giving the palette a label name, and then `set` assigns a palette to a unit for a particular class. (In this case, Colm(`0x9`) when he's a thief(`0xD`), which is his base class)  
Finally, we have the palette that I mentioned before. pal2EA can compress and insert up to four palettes. One for the unit as a player, enemy, ally, or other unit.  

pal2EA has an option to just copy-paste one palette, and just use it for the other factions (Which is what's done in this example).  
If you want to see more details on pal2EA, check out its [README](https://github.com/Teraspark/Pal2EA/blob/master/README.md).

Here's a list of what numbers mean what for setting the units' class palettes (See how that 1 matches Colm's base class 1 option?):
```
0 = Trainee Class
1 = Base Class 1
2 = Base Class 2 (for trainees)
3 = Promoted Class 1
4 = Promoted Class 2
5 = Promoted Class 3
6 = Promoted Class 4 (for Ewan)
```

Now you can run pal2EA on your palette file. It should generate `Palette Installer.event` and `Palette Setup.event`. The installer file is the one that should be included.

## Spell Animations

### Insertion

Before you can insert spell animations, you'll need [Circles' Spell Animation Creator (Or CSA)](https://feuniverse.us/t/fe6-7-8-circles-spell-animation-creator-updated-to-v1-1/1946).

Once you have it extracted and in your buildfile, don't forget to `#include` the `Master Spell Animation Installer.event` file.

Included is a `readme.txt` file with instructions on use. Even though it's there, I'll walk through the process.  
`CSA_Creator.exe` be in the same directory as the `grit` folder and the spell animation script and frames to work.  
With this in mind, it might be a good idea to make a copy of `CSA_Creator.exe` and the `grit` folder, and place them in a separate folder for easier copying.  

You'll need a spell animation script and frames to run CSA on. You can get these from the [Fire Emblem Resource Repository](https://github.com/Klokinator/FE-Repo).  

Once you have the spell animation you want to insert, copy `CSA_Creator.exe` and the `grit` folder to the animation's directory. Then run `CSA_Creator.exe` on the animation script.  
This can be done either through drag and dropping the script onto `CSA_Creator.exe` or using the console.  

You should get a new Installer .event file. Now you can include it in your buildfile (Preferably under the `//animations go here` line in `Master Spell Animation Installer.event`).

Once you do this, open the genrerated installer file. Near the top, you'll find these lines:
```
#define spellanim_index 0x0 //Change SPELL_INDEX to whatever spell you're replacing
setCustomSpell_dim(spellanim_index) //change to "setCustomSpell_nodim" to skip screen dimming
```

The name of the defined index is based on the name of the animation script file.  
Change the value of the index to whatever spell animation ID you want to use.  
If you don't want the spell to dim the background, replace the `setCustomSpell_dim` macro with `setCustomSpell_nodim`.

### Assigning Animations

To assign a spell animation to a weapon, go to open `Tables/Item/FE8 Spell Association Editor.csv`.

Dispite this being a CSV, this file represents a list (not a table).  

NOTE: The following applies if you are not using the Skill System

The first thing you should do is go to the top-left cell and replace the address there with `INLINE SpellAssociationList`.  
`INLINE` is to tell C2EA that this list isn't at a specific address, and `SpellAssociationList` is the label name of the list.  
Doing this will also repoint the original spell association list automatically, so you won't need to do it yourself.  

Next, you need to add a terminator to the end of the list. Copy [this row](https://github.com/FireEmblemUniverse/SkillSystem_FE8/blob/master/Tables/NightmareModules/Items/SpellAssociationList.csv#L163) from the Skill System repository and add it to the end of the .CSV file.

NOTE: Okay, back to where things apply everywhere.

Make a new row right before the TERMINATOR row.  
Now fill in the data you want in each column. The names are inconsistent across the Easy Buildfile and the Skillsystem, so I'll be covering the Easy Buildfile names first:

| EasyBuildfile Name               | SkillSystem Name         | What it does                                                                                             |
|----------------------------------|--------------------------|----------------------------------------------------------------------------------------------------------|
| Weapon                           | Item ID                  | The ID of the item you want to display the spell animation for.                                          |
| No. of Chars to Display (1 or 2) | Chars to Display (1 or 2)| Determines if this is a battle with two people (for combat) or one person (for self-healing)             |
| Ranged Animation to Use          | Spell Animation          | The ID of the spell animation you want to be displayed.                                                  |
| Ranged Animation Enabled         |                          | The second half of the spell animation ID short. Unneeded if you have fewer than 255 animations.         |
| Alternate Pointer                | Map Effect Proc          | Used for special map animations. Should be set to 0 in most cases.                                       |
| Return to original position (map)| Damage                   | Decides whether the combat display and damage flash are used. Set to 1 for weapons or the game will hang.|
| Facing position (map)            | Facing Direction (Map)   | Decides what direction sprites will face during map animations. 0 is used for weapons.                   |
| Enemy's flashing colour (map)    | Color When Hit           | Decides the color the target will flash when hit during a map animation.                                 |
| ##UNKNOWN##                      | N/A                      | Either unused or padding. They'll be fine just being set to 0.                                           |

With this information in mind, you should be able to fill in the data you need to assign your spell animation.
