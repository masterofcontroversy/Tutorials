Buildfiles are an incredibly flexible method of GBAFE hacking. They give you near perfect freedom in what you can do and how you can do it.
Of course with all that freedom, things won't be very easy when you start out. But that's what this guide is here for!

## About this tutorial:

In this tutorial, I'm going to teach you the basics of inserting things into your ROM, while walking you through the process of finding information you'll need and applying it.  
Since there are so many different ways to do things with buildfiles, I'm only going to cover one or two, along with how they work, then I'll let you decide if or how you want to put your own spin on it

## The basics:

Ahh yes, the work folder. This is something that you will be familiarizing yourself with very well.  
Everything involving the building of the hack happens in this folder. Now, you could start from scratch with an empty folder,
or you could use one of the templates for hacking to give yourself a jumpstart. If you're starting, I recommend using Mystic's EasyBuildfile here: https://github.com/MysticOCE/EasyBuildfile

If you're going to start from scratch, these are some common sub-folders that are used:

```
Root
|-- Engine Hacks
|-- Event Assembler
|-- Events
|-- Graphics
|   |-- Animations
|   |-- Mugs
|   `-- Palettes
|-- Map
|   `-- Tilesets
|-- Music
|-- Tables
`-- Text
```

For the sake of this tutorial, I'm going to assume you're using Mystic's EasyBuildfile.

Okay, now that we have that knowledge under our belts, we need to know about what files we need.  
The most important thing for Fire Emblem ROM hacking is an Fire Emblem ROM (FE8 in this case).  
We've all been through this I'm sure, so I'll make this short. I can't tell you where to get a ROM, so you have to get it yourself by legally dumping it. (hear that Nintendo? Now get off my case)

Remember to name the ROM `FE8_clean.gba`

So now that we have our totally legal dumped ROM of FE8, now we can focus on the Build script (Not the build event file).  
This little script normally called something like "MakeHack" automates the process of patching your edits into the ROM while making a copy of the original.  
This is really helpful when you want something to be done before the actual building of the ROM. Like processing tables for example.  

Now for the build event file. If Event Assembler is where the magic happens, the build event file is the spell book to channel that magic.  
From here you get to decide what and where you want things to be. But remember, these are still instructions for a computer,
so you have to play by the rules. What rules? I'll get into that during the "event language" section.


## How to speak Event Assembler:

It's very important to understand how Event Assembler reads instructions. If you put something in wrong, you're going to have a bad time.

Here's some lingo all the cool FE hacking kids are using nowadays

CurrentOffset  
    This is where EA is at the moment. the current offset changes as things are done to the ROM, or if you directly go to a specific address. (More on that later)
    When ever anything is inserted, it's data will always be from the current offset, to however much space it takes up.
    For example, if the current offset was at 0x20(32 decimal), and I inserted something that was 16 bytes in size, the new current offset would be 0x30(48 decimal)
    
### Byte  
Bytes are a form of data. Everything in the ROM is in byte form. I'm not going to go in detail about them right now,
but believe me when I say they're VERY important.  

In Event Assembler, bytes can be read in decimal form (0-255), or hexidecimal form (0x0-0xFF).  
However, when going into the ROM, everything's going to be hex...unless you're editing the ROM as binary or something (Please don't)
    
### Short  
Shorts are pretty much two bytes back to back. If you need to use bigger numbers. Shorts are the next step up from bytes.
    
### Word  
What's better than two bytes? FOUR bytes! Words are four bytes long, so they're also two shorts in length.

### Label  
Labels are used to well, label things. Instead of having to remember where every little thing is, you can just reference labels to tell Event Assembler you want to use what's in that particular label.
The syntax is as follows:
```
Label:
<Things you want in the label>
```

Let's just say that you have a byte of data you want labeled. This is how you'd do it:
```
ALIGN 4
LittleByte:
BYTE 0x64
```
It's very important to keep in mind that labels may not have spaces in them, and a colon must always follow the label's name.

<hr>

Now let's talk about some of the things that you can do with an event file.

### #include
```
#include <File name>
```
This is used to add other event files to the event file "tree". For example, if you had an event file that adds a portrait of Eirika with green hair, and another that Replaces Determination with Ode To Joy, they could be added to the ROM buildfile by "#include"-ing them.  

One very important thing to remember is that if you want an event file to be used, it must either be included in the build event file, or included by a file already included by the build event file.

### #incbin
```
#incbin <.dmp file>
```
"#incbin" is pretty much like "#include", but for .dmp files. It basically just dumps the contents of the .dmp file into the ROM starting from the current offset

.dmp files strictly work on a raw hex basis, so unless you know what you're doing, you shouldn't edit them.

While including .dmp files can be helpful there are times when something called macros can be easier, faster, and more reliable than than using .dmp files. But there are times when .dmp files work best. We'll be looking into that later.

### #incext
```
#incext <executable to run> <Possible arguments> <File to run on>
```
So, this one is not so simple considering how much you can do with it. There are some tools that come pre-packaged with Event Assembler that you can use right from the event files themselves.  
Say I wanted to compress an image with the Png2dmp program, and then include it. I could do that from within an event file by typing: `#incext Png2dmp "myimage.png"`.  

There are many other things you can do with "#incext", but it's better to talk about them when they're relevant.  

One last thing of note: you can run these programs yourself and "#incbin" the resulting .bin files so Event Assembler won't have to do it every time you build your hack. This saves on build time. (I'll demonstrate this later)

### #define
```
#define <New something> <Old something> | <Name(Arguments)> <Things for definition to do>
```
"#define" is used to either define something, or run a set of instructions easily with one command.  
Say I wanted To call Seth something other than `0x02`, I could type `#define Seth 0x02` to do that. The same applies for lots of other things too.
    
The second use I mentioned works something like this: `#define AnimPointer(Type,Type_Designation,AnimID) "BYTE Type Type_Designation AnimID 0"`

This thing right here is what's known as a macro. Macros are used to turn what would be tedious, and repetitious into something neat and managable.  
If you're curious about this macro, it's for animation pointers, but I'll get into that later.

### #undef
```
#undef <Definition>
```
"#undef" reverses what #define does, so the definition no longer exists.

### ALIGN
```
ALIGN <Integer>
```
This is a simple one. Whenever you insert something, it's placed in a hex address that the game looks up when it needs to.  

Trouble is, you can't just have the inserted things be in odd alignments. Generally, it's always a good idea to have your
inserted items be in addressed that are multiples of four. That is what "ALIGN" is for.  

"ALIGN 4" for instance, places enough blank space for whatever that comes after it to start at an address that is a multiple of four. We'll see examples of this later, but trust me when I say "ALIGN" extremely useful.

### BYTE

```
BYTE <byte>
```
Inserts a byte. eg. `BYTE 0x40`

### SHORT
```
SHORT <two bytes>
```
Inserts a short. eg. `SHORT 0x1800`

### WORD
```
WORD <four bytes>
```
Inserts a word. eg. `WORD 0xDEADBEEF`

### POIN
```
POIN <address>
```
Pointers are very important since they well, point to things. There are sometimes when using a direct address just won't do.  
In normal cases (Or if you're changing the game's code) pointers are used to tell the game where to read data.  

Say you had Seth's data. The game would need to check it constantly, so instead of putting it in different places, it's stored in one location that every function can look at after they point to Seth's data.  
Kind of like a library. Instead of having copies of all the books in everyone's house, they're stored in one place for everyone to find.

### ORG
```
ORG <address>
```
Say you want to jump to a specific place in your ROM. ORG is how you'd do it. ORG jumps to the specified place in the ROM, and any data you add will be inserted starting from that location.

### PUSH
```
PUSH <address>
```
One thing you might have noticed with ORG is that there's no way to go back to where you jumped from. That's where PUSH, and its partner POP come in.
PUSH saves the current offset for you, so you can go back to it later.
    
### POP
```
POP <address>
```
POP on the other hand, sends you back to the offset that was PUSH-ed.

### MESSAGE
```
MESSAGE <text>
```
This outputs a message consisting of what you want. You can even put definitions in here so you can see what's what when troubleshooting.

### WARNING
```
WARNING <text>
```
Does the same as MESSAGE, but displays itself as a warning instead.

### ERROR
```
ERROR <text>
```
Does the same as the last two with one twist; If you get an error, Event Assembler stops. This is very nice to make your own errors to suit your own needs.

## Editing Text:

Okay let's start actually changing things in the ROM. I'm sure you're no stranger to text (If you are, how are you reading this?).  
Text editing is a very important part of making hacks. Without it, you'd just be stuck with defaults. And defaults are disgusting.

Head on over to the text folder, and take a look at what we have:

-textprocess:  
    This is the Event Assembler of text. It takes the text buildfile as an input, and puts out all the things necessary for inserting text into the ROM  

-text_buildfile:
    This is the ROM build event file of text. textprocess uses it to figure out what text you want, and where you want it to be.  

-install text data:  
    This is the installer made by textprocess. For your text to be inserted, you need to include it. There isn't really a reason to touch it by hand since text process handles it.  

-text definitions:  
    Textprocess makes this to go with text install data so Event Assembler can actually use them.  

-_textentries:  
    This is where the text you typed is placed after it's been handled by textprocess.  

-ParseDefinitions:  
    The parse definitions are the "dictionary" of codes that you can use to do things like load character portraits, and change how the text is presented.  

<hr>
    
Okay, now that we know a bit of what these do, let's change some things around, shall we?

Open up the text buildfile, and take a quick look. See that top line? It's very important.  
The `# 0x0903` (The space is important) is telling text process that this text should go in textID 0x0903.  
Ignoring the next few lines, there's something else.

`##` tells text process that the following text should go in the next textID. 0x0903 was the last textID used, so the next text inserted will be placed in 0x0904.

One thing to note is after either of these on the same line, you can put in a space, then the label of the text, so
`# 0x0903 TextLabel1`
or
`## TextLabel2`

These Labels may not have spaces in them, so no `## Text Label 1`. Underscores are allowed though.

Now I could explain how all this text works, or I could point you in the direction of this text editing tutorial by Darrman
https://feuniverse.us/t/the-ins-and-outs-of-text-editing/6820

Was that a good read or what? (You did read it didn't you?)

You might have noticed that Roshea has hijacked Eirika in the hacked ROM. Sadly, he's still labeled as Eirika. Let's fix that.  
First things first, we're going to have to find where Eirika's name textID is.

Let's go into the text dump in the Text folder (It's inside of "Useful References" if you're using the easy buildfile)  
Since we know that we're looking for Eirika's name all by itself, let's search for `Eirika[X]`

Now we should have the textID needed. Head over to the text buildfile, and put the name "Roshea" (Or whatever you want really) in Eirika's name textID.

...Great. Now run MAKE HACK_textonly.cmd, and load up the ROM.  
Now Roshea won't have to suffer from an identity crisis.

If you want, there's another way to change Eirika's name.  
You can go to the character editor table (the .csv file), and change Eirika's name value to the label of the text you want. Easy peasy.  
Just be sure it's not too long.
