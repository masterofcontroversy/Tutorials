Graphics (Dun, dun, Dunnnnn)

So you made it here huh? Can you handle the weight of inserting graphics?
Inserting graphics isn't really that hard. In fact, it's quite easy once you get it down.

## Portraits:

Go over to Graphics/Mugs You see that png file? Feel free to take a look at it, but we're going to use something else.
Find a portrait that you want to insert (The FEU mugging blitzes, and the ultimate graphics repository are great sources), and put it in the Mugs folder.  
Now open up `Mug Installer.event` in your text editor of choice.  
Underneath the `Roshea` label, make a label of your own for your portrait. (Remember. No spaces)  
Now we need to use one of the included tools to compress the .png file. That tool would be PortraitFormatter. Remember what to type when we want to include things right after compressing them?

```
ALIGN 4
YourMug:
#incext PortraitFormatter "yourmug"
```
Great! Now that the portrait is in there, we just have to assign it so it can be used.
Time to disect a macro. This is the setMugEntry macro: `setMugEntry(0x2,YourMug,2,5,2,3)`

-The first argument is `0x2` this is the slot that this mug will be using.  
-"YourMug" is of course the label that you just made, meaning that what's under it (the mug itself) will be used.  
-The next two arguments (2,5) are the coordinates of the mouth. The game needs this information for the mouth animations.  
-The last two arguments (2,3) are the same thing as the mouth, but for the eyes.  

Now run the makehack file, and take a look at the character that uses the portrait you replaced.

There we go! You have inserted your mug! Now for...

## Combat Animations:

Vanilla animations are pretty cool, but custom animations are also cool, and I'm gonna tell you how to insert them.

First, you'll need [Animation Assembler](https://feuniverse.us/t/fe7-8-animation-assembler-convert-feditor-format-for-insertion-with-ea/1880). It's already included in `Graphics/Animation` if you're using the easy buildfile. (The file itself is AA.exe)

Next, you'll need the animation you want to insert. The animation folders hold a lot, so I'll quickly explain what they contain.

-Animation frames: Combat animations are done in a frame-by-frame style. These are the frames that are used for that animation.  
-Animation sheets: Same as the animation frames, but in a format that when compressed, the game can use.  
-Animation script: These are used to tell the game what frames of animation to use when, and where. The human readable version in a text version, and there's a bin version. We wan't the .bin version.  

If you're missing the .bin file, then you can open up FEBuilderGBA, and insert the animation using the .txt script, then export the animation as a dmp.

Let's use the the [beta Eirika animations.](https://github.com/Klokinator/FE-Repo/tree/main/Battle%20Animations/Lords%20-%20Vanilla%20and%20Custom/%5BFE8%20Eirika-Base%5D%20%5BF%5D%20T1%20Beta%20Eirika%20Fixed%20by%20Jono%20the%20Red)  
You can add any valid animation for most classes, but for the sake of this tutorial, I'll be using something we can see instantly.

Run Animation Assembler on the .bin file of your animation. This can be done either by dragging the .bin file and dropping it on AA.exe, or by using the command prompt. <!-- (If you're using the latter, you should already know how to do that) -->

After Animation Assembler finishes, you should have a shiny new Installer.event file. Open it up and change this line:
```
AnimTableEntry(0x0) //CHANGE THIS TO THE SLOT YOU ARE REPLACING
```

In this case for Eirika, we want to change it to `0x3` for her sword animation, and `0x4` for her unarmed animation.

Now open the `Master Animation Installer` file and include the Installer.event files. One thing though...

Now those labels in the rest of the file? Those are based off of what the .bin is named. This is important because we can't have labels with the same name or else we'll get errors. If we happen to install an animation .bin with the name `Sword`, the labels would match Eirika's sword animation labels.

One way to fix this is to rename the .bin file something that you probably won't be naming your labels.  
Another option is to go into the master animation installer file, and include your animation installer files nested in curly braces like so:

```
{
    #include "YourAnimation.event" //Labels are Sword
}
{
    #include "YourOtherAnimation.event" //Labels are exactly the same
}
```

Whatever method you use, you need to open the master animation installer file, and include the .event file that Animation Assembler made.

Great, now start up that makehack file, and have Eirika fight something.


## Animation pointers:

I know what you're thinking. "What if I want to do something like insert an animation for a paladin using an axe?" Don't worry. I've got you covered.

The first thing we need to do is understand animation pointers.
This is the animation pointer for Ephraim's Lord class:
```
01 01 01 00 09 01 02 00
```

Time to dissect.

The first byte can be used for one of two things. The first is the weapon type being used, and the second is a specific item.
The Weapon type decides what type of weapon will trigger the animation. These can be:
```
00 = Sword
01 = Lance
02 = Axe
03 = Bow
04 = Staff
05 = Anima
06 = Light
07 = Dark
09 = Item (Unarmed)
0B = Monster Weapon
0C = Ring (For Dancers)
11 = FireStone
```

The second byte, which I'll call the "Type Designation" is for whether you want all items of the same type to share an animation, or have the animation play for one item only (Commonly used for throwing axes). The options are:
```
1 = all items of that type. (eg. All swords)
0 = Specific item.
```

The last two bytes (a short if you will) are for the animation ID.

And finally once it's done it uses four bytes of `0` at the end.

So looking back at Ephraim's Lord class animation pointer you can see the first word (Four bytes, remember?) basically says:
```
This is for lances, This is for all the lances, use animation ID 1
```
Now take a guess at what the second word means.

Okay, now that we know all that, let's give paladins axes.

First, download the [paladin axe, and handaxe animations.](https://github.com/Klokinator/FE-Repo/tree/main/Battle%20Animations/Mounted%20-%20Cavs%2C%20Paladins%2C%20Rangers/%5BPaladin-Base%5D%20%5BM%5D%20Vanilla%20%2BWeapons)

Run them through Animation Assembler, and replace the bonewalker's animations for now. (A2, and A3)

We'll still be wanting to use the vanilla paladin's animations since we're just adding axes. Here are the animation IDs for them:
```
#define PaladinSword 0x3A
#define PaladinLance 0x3B
#define PaladinUnarmed 0x3C
```

Next we'll need the throwing axe item IDs. A quick look in `Event Assembler/EA Standard Library/FE8 Definitions.txt` tells us...
```
#define HandAxe 0x28
#define Tomahawk 0x29
#define Hatchet 0x2C
```
(Don't forget you can use the definition names instead of the numbers)


Event Assembler has some macros to help with making animation pointers. you can find them in `Event Assembler/EA Standard Library/Animation Setters.txt`

I also made my own macro for this: `#define AnimPointer(Type,Type_Designation,AnimID) "BYTE Type Type_Designation SHORT AnimID"`

These do the same thing as BYTE-ing everything in would, only it's easier to read.

Now make an event file for your animation pointers. (It can be anywhere you want, just make sure it's easy to remember)  
Open it up and make a label for your animation pointer. (Remember to ALIGN 4)  
Now input the information for your animation pointer.

Using what you just learned, make the animation pointers for the vanillia paladins plus axes.  
Remember to put `WORD 0 0` or `EndClassAnimation` at the end.

<details>
<summary>It should look something like...</summary>
<br>

```
ALIGN 4
PaladinAnimPoint:
AnimPointer(Swords, 1, 0x3A)
AnimPointer(Lances, 1, 0x3B)
AddHandAxeAnimation(PaladinHandaxeAnim) //Nifty handaxe animation pointer macro
AnimPointer(Disarmed, 1, 0x3C)
EndClassAnimation
```
</details>

Now we need to tell the game to actually use this pointer.  
Go to the class table, and find the class you want o change the animation pointer of, then go to the `Battle Animation Pointer` column. Then, in that cell type your label name with `|IsPointer` on the end.  
While you're here, give Paladins a default axe rank of at least 1. We need this so Paladins can use axes.

You can also use EA's `SetClassAnimation(ClassID,Pointer)` macro. It orgs to the animation pointer part of a class' data and points to a new animation pointer. If you use this method, you'll have to do it before the tables are included. (The tables overwrite the data)

Go to the ExampleChapter file, and give Seth an axe, and a handaxe (Make sure he's a paladin)  
(Since the ExampleChapter doesn't have Seth in it, you can just paste the following under the `Units` label)
```
UNIT Seth Paladin 0x0 Level(2,Ally,0) [5,7] 0x0 0x0 0x0 0x0 [IronAxe, HandAxe] NoAI
```

(Don't get rid of the final `UNIT` at the end. It's to mark the end of the unit list)

Run makehack (fully, so the tables will be updated), and try out Seth with an axe.

Good? Good. On to the next thing.

Palettes (Ughhh):


Map Sprites:

Okay, now this one is a bit of a doozy
t
