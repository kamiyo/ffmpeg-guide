A hopefully beginner-friendly guide to using ffmpeg, with some terminal tips.

## The Software: ffmpeg

https://ffmpeg.org/download.html

This is a multipurpose video/audio/image processing tool. The unfortunate thing is there's no GUI (graphical user interface) for it, so you have to do everything on the command line. That's what scares people most, but hopefully I can help you a bit.

Download the correct Windows or Mac version for your computer and install it:

windows: https://github.com/BtbN/FFmpeg-Builds/releases/download/latest/ffmpeg-n5.0-latest-win64-gpl-5.0.zip

mac: https://evermeet.cx/ffmpeg/ffmpeg-5.0.1.zip

Save the file, and then extract it. Remember where you extracted the file.

## Command line / Terminal basics

Windows: In your start menu, type `powershell` or `pwsh` and click on the first thing that shows up.

Mac: In spotlight search, type `Terminal`, and run that app.

These tips are common in all terminal-type apps.

 * Press `Enter` to run whatever you have typed (I'll tell you what to type later)
 * Press the up arrow key to scroll through your command history (if you're looking for something you typed previously, or you want to run the same command but change some things).
 * If you're trying to find a file or directory, you can press `Tab` to cycle through possible autocompletes.
 * If something seems stuck, or you want to stop whatever it's currently processing, use `Ctrl + C` (feel like you are copying something, but it's a different behavior).

Some commands to remember.

* `pwd` is "print working directory"

    This tells you what folder you are currently in.

* `ls` is "list files"

    You can try this immediately upon opening your terminal. If you type `ls` and then enter, it will show you what files are in the current folder you are in.

* `cd` is "change directory"

    For example, if there's a folder named "My Folder" in your current directory, you can go to that one by typing `cd "My Folder"`. This is a warning about spaces. If your folder or file name has spaces in it, surround the entire name with quotes.
    To move back up to the previous folder, you should do `cd ..`. You can go into nested folders, or go up multiple folders by chaining it, examples:

    > Current directory: `C:\Users\Sean\Pictures`  
    Desired directory: `C:\Users\`  
    Command to run: `cd ../../`  
    Note: on Windows, \ and / should both work. On Mac, use /  
    Note: last / or \ are not needed, but I type it out of habbit.

    > Current directory: `/Users/Sean`  
    Desired directory: `/Users/Sean/Downloads/ffmpeg`
    Command to run: `cd ./Downloads/ffmpeg`
    Note: the leading `./` is for safety, but it isn't technically necessary. It means "in the current folder"

    Demonstrating Tab autocomplete:

    > type: `cd ./Do` then press Tab.  
    It should autocomplete to: `cd ./Documents`. But we want Downloads, so we press Tab again.  
    Then it should display: `cd ./Downloads`. This is what we want, so then we press Enter.

## Let's go

Okay, so in terminal, let's go find where we put `ffmpeg` (for macs) or `ffmpeg.exe` (for windows). 

This example is for Windows:

Let's say I extracted it, and it put it into `C:\Users\Sean\Downloads\ffmpeg-n5.0-latest-win64-gpl-5.0`. These are my steps:
1. `pwd` and enter: This shows me `C:\Users\Sean`. Great, so I know I need to do:
2. `cd ./Downloads/ffmpeg` and then press Tab because I don't want to type all that stuff after it out. It should autocomplete it for me to `.\Downloads\ffmpeg-n5.0-latest-win64-gpl-5.0`
3. What's in this folder? I type `ls` to see. Oh, there's another folder with the ffmpeg name? Let's go into that one too:
4. `cd ./ffmp` + Tab, then Enter
5. What's in this folder? type `ls`. Oh, I see `bin`, `doc`, and a `LICENSE.txt`. The `ffmpeg.exe` should be in `bin`:
6. `cd ./bin`, enter, and then check the folder contents with `ls`. Aha! I see ffmpeg.exe. Okay.

This was just to walk you through how you would sort of look for something if you didn't know quite where it was. But now you know, you can consolidate the steps to this:
1. `cd C:\Us` + Tab + `Sean\Dow` + Tab + `ffmp` + Tab + `ffmp` + Tab + `bin` + Enter.

For Mac, similar:

1. `pwd` + enter: shows me `/Users/Sean`
2. `cd ./Downloads/ffmp` + Tab
3. `ls`: shows `ffmpeg`. If you're not sure whether it's a file or folder, you can try `cd` into it, and if it fails, you know it's a file.

## Try running ffmpeg

Now that we're in the folder for `ffmpeg`, try running it:

`./ffmpeg`.

It should spit out a bunch of info that's not important. If its says something like "command not found" or something, then you're not in the right folder.

## Using an input file

So you have that video file you want to edit. Let's say it's `C:\Users\Sean\My Videos\video.mp4`. I can first try to open it with `ffmpeg`

`ffmpeg -i "C:\Users\Sean\My Videos\video.mp4"`

the `-i` tells ffmpeg to use the file following it as input

Remember, use Tabs when trying to type out the video file location. It'll help you tremendously. If tab doesn't do anything, then either you typed something wrong or there's nothing in the folder.

See if that spits out any info. It shouldn't do anything to the file yet.

## Understanding parameters

In command line programs, you pass options and parameters using flags. They look like `-i` or `-e` or `-someparameter`. Usually, after that flag, you can type in the actual value, for example `-i input.mp4` or `-c:a libmp3lame` or `-crf 19`. In the case of `ffmpeg`, the output file is the last thing you type.

## Try to sync audio and video

Okay, so let's say you think the video is like maybe a tenth of a second (100 milliseconds) behind the audio. We need to do this:

`ffmpeg -i "path/to/input/file.mp4" -itsoffset 0.1 -i "path/to/input/file.mp4" -map 0:v -map 1:a -c copy "result.mp4"`

Concept:
1. We're going to give ffmpeg two input files, and have it mix it together, except in this case, the two input files are the same one.
2. We're going to take the video from one file, and the audio from the other file delayed, and then mix them together.
3. This should fix the video being delayed (because we're delaying the audio now).

Command explanation:
* First look at the part that says `-map 0:v -map 1:a`. This means we're mapping the first input video to the output video, and the second input audio to the output audio. In computers, often times counting starts at 0, so 0 refers to the first input, and 1 refers to the second.
* Since our video is delayed in the original file, that means we need to delay the audio, so it becomes sync'd up again.
* In order to delay the audio, we're going to apply a timestamp offset to the second input (remember `-map 1:a` means second input goes to the resulting audio).
* The delay happens with `-itsoffset` which stands for input timestamp offset, which causes the subsequence `-i file` to be loaded in delayed.
* `-c copy` tells ffmpeg to not do any converting, it's just going to re-mix the audio and video together.

Testing:  
Maybe our delay is not quite right. How do we test it? Well, we can try the command on a small segment of video. If we want to only try 10 seconds, we can add `-t 10` to right before the output file name:

`ffmpeg -i "path/to/input/file.mp4" -itsoffset 0.1 -i "path/to/input/file.mp4" -map 0:v -map 1:a -c copy -t 10 "result.mp4"`

Then you can play the `result.mp4` and check it. If it's still delayed, increase your `-itsoffset 0.1` to maybe `-itsoffset 0.15` and adjust. If it's too much the other direction, lower your offset.

Remember, use the <b>up arrow key</b> to get the last command you ran. Then you can use your left and right arrow keys to go back and change some of the parameters. That way you don't have to retype EVERYTHING again.

If the audio is the one that is delayed, you have two options:
1. you'll have to switch the map portion to: `-map 0:a -map 1:v`.
2. or you can use a negative offset `-itsoffset -0.05`.

## Have fun
`ffmpeg` has SO many possibilities. You can take a look at https://trac.ffmpeg.org/wiki and scroll down to the community contributed documentation, or the official documentation page: https://ffmpeg.org/ffmpeg.html.

Finally, Google and Stackoverflow are your friend.