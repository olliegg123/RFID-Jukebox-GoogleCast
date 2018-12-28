<h1>RFID Jukebox Guide</h1>
How to setup add an album, for the purposes of this, I will be providing examples to download X Infinity by Watsky and my device has an ip of 192.168.1.189.

<strong>TLDR:</strong> Setup magic-cards (https://github.com/maddox/magic-cards) to call a script containing a go-cast (https://github.com/barnybug/go-cast/blob/master/README.md)command to stream locally hosted music to a google cast device or group.

<h2>Preface</h2>

My dad saw the video by <insert Redditor name here> on the 23rd December with us flying to Sydney on the 26th. So I decided I was going to sort it/something that worked the same for us. So I apologise that this is not at all an efficient or even a close to impressive method of doing this. Due to us not having Google Play Music, that was out of the question. So here is how I did it. It’s not ideal, but I can scan a card and it plays music sooooooooo....
Also this is the first instruction set I’ve ever made for programming something so I’m sorry if it sucks. I’ll create a discord so you can ping questions and if I can help I will or others might be able to. Here’s a link: https://discord.gg/tK6n3hh

<h2>Introduction</h2>

First off I’m going to go over how this looks and how it all fits together in a kind of diagram format:
<br><br>I promise I'll actually make a diagram of this one day<br><br>
Card —(scan)—-> RFID reader —-(USB)—>Pi—(cards.json)—->identify card (magic-cards) —-(actions.json)—>call script—-(go-cast)—> call locally hosted music (Apache)  —-(go-cast)——> Output on Google Cast Group

<h2>Hardware and Basic Pi Setup</h2>

<h4> Hardware</h4>
So the hardware I used was:
<ul>
<li>Raspberry Pi 3 (Might work on older models or even Pi Zero (W), who knows. Try it, let me know)</li>
<li>16GB SD Card (Depends on your preference but at least 8GB but you’ll be storing music on this if you follow my instructions)</li>
<li>Official Pi 3 power supply (I’m fancy okay)</li>
<li>125KHz RFID reader - (Amazon)</li>
<li>125KHz RFID cards  - (Amazon) </li></ul>

<h4> RFID Reader Config</h4>
First of all you need a windows desktop to configure the RFID reader follow these instructions from the magic-cards (https://github.com/maddox/magic-cards)repo:<br><br>
1. Download [this software](https://www.dropbox.com/s/ena4ukh9wewhj9x/rfid-reader-programmer.zip?dl=0) on a Windows PC or Laptop. <br><br>
2. Plug the RFID reader into the Laptop or PC USB port and open the software.<br><br>
3. Make sure that the buttons shown in the image below are selected, then click <code>set</code>.<br><br>

![Programmer Setup:](https://github.com/maddox/magic-cards/blob/master/docs/images/card-programmer.png)
<br><br>
Now the reader is good to go for the Pi, but it might be worth leaving it plugged into the Windows device for future steps. <br>(Pro-tip form magic-cards: Buy 2 RFID readers so that you can have one connected to the Pi and one connected to a Windows PC to enable creating cards to be easier.)

<h4> Pi OS Setup </h4>
You should have the basics setup, I used the latest raspbian image flashing it onto the Micro SD using etcher. I used the raspbian with desktop and essential programs but I don’t think it’s necessary really, I’d recommend the one with a desktop just because it helps with debugging network problems and stuff.

<h4> Pi Network Setup </h4>
Make sure to either use it through Ethernet or to put a wpa_supplicant.conf file in the boot with the necessary config inside (check this link for more info) or even use the graphical interface to configure the WiFi.

<h4> Pi SSH Setup </h4>
Also in the boot drive put a file just called ‘ssh’ with no file or extension or anything, this will enable ssh on first boot and allow you to connect.

If you’re on Mac or Linux you can use the standard terminal but for Windows you’ll need to download putty (putty link). Find the up of the Pi from the router or from the gui and then ssh to it. There are many guides online about this. Here’s [one](https://raspberrypi.org/documentation/remote-access/ssh/).

<h4> Pi SFTP Information</h4>

For all file transferring I would recommend using a program with an interface to easily transfer from your PC/Mac to the Pi, such as CyberDuck with SFTP.

<h2>Project Setup and Installing</h2>

<h4> Node, NPM, Yarn and Git Install </h4>
There's a set of instructions on the magic-cards install page which you can use instead of these.<br><br>
So make sure you have git installed: <code>sudo apt-get install git</code><br><br>
I can't find the exact commands I used to install node, but I know I used the nvm manager and had to use the latest version of node (9.11.1 at the time of writing this. But just check online for how to install node, then run the next command. I'll update this if somebody finds commands which work exactly).<br><br>
Then install yarn:<code>npm install yarn -g</code>


<h4>Magic-Cards Setup</h4>
Check out this first repository (magic-cards) and see all the cool stuff, but I’ll copy the instructions across here for you to follow:
<br><br>Navigate home:<code>cd ~</code>
<br>Clone the repo:<code>git clone https://github.com/maddox/magic-cards</code>
<br>Navigate into the folder:<code> cd magic-cards</code>
<br>Run the setup script:<code> script/setup</code>
<br>Run the install script:<code>script/install</code>
<br>Run the stop script:<code>script/stop</code>

<h4>go-cast Setup</h4>
Another great repository to check out (go-cast) and this allows casting of media to the chrome cast devices. I’ll post instructions below again:<br><br>
Visit [this page](https://github.com/barnybug/go-cast/releases/tag/0.1.0) to find the version you need, from a Pi it is the<code>cast-linux-arm</code>.

<br>Download your file (cast-Linux-arm for a pi): <code>wget (link to download from page above)</code><br>
<strong>E.G.</strong><code>wget https://github.com/barnybug/go-cast/releases/download/0.1.0/cast-linux/arm</code><br>
<br>Move the file:<code>sudo mv cast-my-platform /usr/local/bin/cast</code><br>
<strong>E.G.</strong><code>sudo mv cast-linux-arm /usr/bin/local/cast</code><br>
<br>Edit the permissions:<code>sudo chmod +x /usr/bin/local/cast </code>
<br><br>Cast is now setup, check out the repo (https://github.com/barnybug/go-cast/blob/master/README.md) for usability and features etc. It’s really cool and powerful.

<h4> Web Server Setup</h4>
Setup the web server, these instructions are pulled from the [Raspberry Pi Website](https://raspberrypi.org/documentation/remote-access/web-server/apache.md)<br><br>

Install apache:<code>sudo apt-get install apache2 -y</code>
<br>Test it from any device on the network: <code>http://local_ip_of_raspberry_pi/</code>
<br><strong>E.G.</strong> <code>http://192.168.1.189/</code>
<br>Make a folder to store the music:<code>sudo mkdir /var/www/html/music/</code>
<br>Edit permissions to allow easy interaction (probs super unsafe but it's local so idgaf): <code>sudo chmod -r 777 /var/www/html </code>
<br><br>Your html directory is now here: <code>/var/www/html/</code>
<br>And your music directory is now here: <code>/var/www/html/music/</code>

<h3> DISCLAIMER: </h3>
<strong>I do not condone breaking the law or illegally downloading YouTube music if it's against copyright but here is a purely educational way to do it and a really powerful tool to do it with and to check out [ youtube-dl ](https://github.com/rg3/youtube-dl/blob/master/README.md) once again, instructions below for install but read up on its usage because I’ll be giving v simple usage that I’ve done for this project.</strong>

<h4>[youtube-dl](https://github.com/rg3/youtube-dl/blob/master/README.md) setup</h4>

Download the repo: <code>sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl</code>
<br>Install it: <code>sudo chmod a+rx /usr/local/bin/youtube-dl</code>

<h2>Configurations</h2>
<h3>[Magic-cards](https://github.com/maddox/magic-cards) configuration</h3>

Now go to the[ magic-cards ](https://github.com/maddox/magic-cards)config directory:

In here you’ll need 3 files: actions.json, config.json and cards.json. I’ll go into detail about how to make these and I’ll be posting mine as examples. (All this can be found on the[ magic-cards ](https://github.com/maddox/magic-cards)repo but I’m just going to explain it how I found it)

<h4>config.json</h4>
 This file defines the usb event to use for the reader as well as the room and Spotify shit if you want it but I didn’t care for it, like I said check out the magic-cards (https://github.com/maddox/magic-cards) repo it’s probably way more powerful than I’m using it for.
 <br><br> For this, copy my config file and ensure you use the correct event number which can be found by doing:
 <br>Navigate to the folder: <code>cd /dev/input/by-id/</code>
 <br>List the devices:<code> ls -l</code>
 <br>Then you should see an output like: <br><code>total 0<br>
lrwxrwxrwx 1 root root 9 Dec 25 16:26 usb-SEM_USB_Keyboard-event-if01 -> ../event2<br>
lrwxrwxrwx 1 root root 9 Dec 25 16:26 usb-SEM_USB_Keyboard-event-kbd -> ../event1<br>
lrwxrwxrwx 1 root root 9 Dec 25 16:26 usb-Sycreader_USB_Reader_08FF20150112-event-kbd -> ../event0</code>
<br><br> Here the event I would be using is<code>event0</code> because the description refers to the reader.

<h4>actions.json</h4>
This holds the possible actions to be taken on the scan of a card, so you can do Sonos stuff if you read the magic-cards repo but I’m just going to be calling scripts, each action named after the album/artist and each script named something similar but ending with .sh.
<br><br>We'll be editing this file later on in the full run through.

<h4>cards.json</h4>
This is auto created if you use the online interface to create cards, but we’ll come onto that later because I had problems with the online interface so I edited the actions.json file directly.

My examples for these files are in this repo.

<h3> Full Config Run Through </h3>
Now basically, I’m going to run through adding one card from beginning to end including (the possibility and instructions pls refer to the disclaimer above) downloading the music.

<h5> Remember:</h5> I'm using a Watsky album for my examples and the local ip address of 192.168.1.189
<br>

<ol><h5><li>Download song/playlist:</h5>
<strong> If you already have the music downloaded you can skip the youtube-dl stuff and just place the songs into the folder stated. Or you might need to 'cat' them if they're individual tracks.</strong><br><br>
So find the album you would like on youtube, either as one song long (heckin easier) or as a playlist which we will concatenate. But I understand that the easy method isn't always possible so I'll show you both dw).
<br><br>
(Also this technique sucks, I know, and I'm very open to ideas which doesn't mean that I have one super long song but like I said in the preface, I had little time and I will improve in the future, but it works.)

   <h5>1a. As a single super long song that is the full album</h5>
   Download the song with this command directly to the hosted folder:<code>youtube-dl --extract-audio --audio-format mp3 -o /var/www/html/music/(song title).mp3 (youtube link)</code>
   <br><br><strong>E.G.</strong><code>youtube-dl --extract-audio --audio-format mp3 -o /var/www/html/music/xinfinity.mp3 https://www.youtube.com/watch?v=2kzI9ksEjx0&t=2846s </code>
   <br><br>
   <h5>1b. As a playlist and then how to concatenate the songs into a single track</h5>
    1. Make a new directory in the music folder: <code>sudo mkdir /var/www/html/music/(artist name)</code>
      <br><strong>E.G.</strong><code>sudo mkdir /var/www/html/music/watsky</code><br><br>
    2.       Download the playlist: <code>youtube-dl --extract-audio --audio-format mp3 -o /var/www/html/music/(artist name)/'%(autonumber)s.%(ext)s' (youtube playlist link)</code>
        <br><strong>E.G.</strong><code>youtube-dl --extract-audio --audio-format mp3 -o /var/www/html/music/watsky/'%(autonumber)s.%(ext)s' https://www.youtube.com/watch?v=qaDuk0qna5g&list=PLblxhEQYeadH27CgeioDeuYgCpu9SbVbj</code><br><br>
    3. Wait for all the tracks to download correctly, the above command will number them all from 00001.mp3 -> 00XXX.mp3 depending on how many tracks there are, this makes the next step easier.<br><br>
    4.       Go to the folder:<code>cd /var/www/html/music/(artist name)</code>    <br><strong>E.G.</strong><code>cd /var/www/html/music/watsky</code><br><br>
    5.       Concatenate the songs:<code>cat 00001.mp3 00002.mp3 00003.mp3 00004.mp3 ...... > (song title).mp3</code><br><br>
      <strong>E.G.</strong><code>cat 00001.mp3 00002.mp3 00003.mp3 00004.mp3 00005.mp3 00006.mp3 00007.mp3 00008.mp3 00009.mp3 00010.mp3 > xinfinity.mp3</code><br><br>
    6.       Move the song to the main folder: <code>mv (song title).mp3 /var/www/html/music/</code>    <br><strong>E.G.</strong> <code>mv xinfinity.mp3 /var/www/html/music/</code><br><br>
    7. Navigate to the music folder: <code>cd /var/www/html/music/</code><br><br>
    8. Clean up by removing the artist folder: <code>sudo rm -r (artist name)</code><br>
    <strong>E.G.</strong> <code>sudo rm -r watsky</code>

<h5><li>Direct yourself to it to make sure it actually is there:</li></h5>
Open up a browser on any device that is on the same network and head to this address to check that the web server is running correctly and the .mp3 is accessible: <code>http://local_ip_of_raspberry_pi/music/(song title).mp3</code><br>
<strong>E.G.</strong> <code>http://192.168.1.189/music/xinfinity.mp3</code><br><br>
You should be able to click play or something and have the track playing through your device. if you've made it this far, I guess my instructions don't suck that much. Make sure to note down this address too for the next step.
<h5><li>Create .sh file with cast to device with url (remember :80 and http):</li></h5>
Okay now onto creating the script file which will be called when the card is scanned.<br><br>
  1. First of all we need to find out the EXACT name of the device we are going to be casting to. To find this run the command: <code>cast discover</code><br>
  This should produce an output of the Google Cast devices which have been detected on the network, groups too. make sure to note down exactly what is within the quotation marks. (Sorry, I'm doing this from memory) but this will be something like: <code>"Kitchen Speaker"</code> for me. So the thing to remember is <code>Kitchen Speaker</code>.<br><br>

  2. After finding out the device name navigate to the config folder of the magic-cards repository you cloned. For me, I would run: <code>cd /home/pi/magic-cards/config</code><br><br>

  3. When here, create a .sh file with the name of your artist or album preferably: <code>sudo nano (album or artist).sh</code><br>
  <strong>E.G.</strong> <code>sudo nano xinfinity.sh</code><br><br>

  4. When in this file you want to write the command to run to trigger the playing on your device. Before you make this file, feel free to test this command standalone to check if it works. But the command to put in the file is: <code>cast --name "(Name of Device or Group of Chromecast)" media play http://local_ip_of_raspberry_pi:80/music/(song title).mp3</code><br>
  <strong>E.G.</strong> <code>cast --name "Kitchen Speaker" media play http://192.168.1.189:80/music/xinfinity.mp3</code><br><br>

  5. Exit and save the file: <code>CTRL+X</code> then <code>CTRL+Y</code><br><br>

  6. Modify the file so that it can be executed: <code> chmod +x (script file name).sh</code><br>
  <strong>E.G.</strong> <code>chmod +x xinfinity.sh</code><br><br>

  7. At this point, I would test that the script works as intended. You should be able to run this command to have the desired output: <code>./(Script Name).sh</code><br>
  <strong>E.G.</strong><code>./xinfinity.sh</code>
<h5><li>Edit actions.json to have the script called being possible:</li></h5>
This step will allow the actions.json file to call upon the script when the necessary card is triggered, it's a pretty simple one, but the actions.json file is very powerful if you want it to do more things, check it out on the magic-cards repo (https://github.com/maddox/magic-cards/blob/master/docs/actions.md)
<br><br>
1. Open up/create the actions.json file: <code>sudo nano /home/pi/magic-cards/config/actions.json </code><br><br>
2. It'll probably be entry, unless you've copied mine in, which has a load of entries. Basically to customise your actions, you need to make sure each enter looks like this:
<code>
                "(ActionName))": {
                  "type": "script",
                  "filename": "(script name).sh"
                }
</code>
<br><strong>E.G.</strong><br>
<code>
                "InfinityScript": {
                  "type": "script",
                  "filename": "xinfinity.sh"
                }
</code>
<h5><li>Start [m</li></h5>
1. Navigate to the [m folder: <code>cd /home/pi/magic-cards</code><br><br>
2. Run the command to start it running: <code>script/restart</code>
<h5><li>Edit cards.json:</li></h5>
There are two ways to add Cards:
<br><br><strong>  6a. Add the Card to the cards.json file:</strong>
<br><br>  1. Edit the cards.json file: <code>sudo nano /home/pi/magic-cards/config/cards.json</code>
<br><br>Use the format below to add the correct details (Once again, check my cards.json file to see):
<code>
                {
                  "code": "<RFID Code>",
                  "type": "album",
                  "action": "<action name>",
                  "artURL": "<link to album art>",
                  "title": "<album title>",
                  "subtitle": "<artist>",
                  "uri": "",
                  "id": <id>
                },
</code>
<br><br><strong>E.G.</strong> 
<code> 
                {
                    "code": "090033639a",
                    "type": "album",
                    "action": XInfinityScript,
                    "artURL": "https://preview.redd.it/7nlnklgq1zgx.png?width=1024&auto=webp&s=bb1110b519e0f5869d8912d1b9f462105b67c92c",
                    "title": "X Infinity",
                    "subtitle": "Watsky",
                    "uri": "",
                    "id": 5217263514
                 },
</code>

<br><br><strong>RFID Code</strong> - This is the ID of the card and is crucial in the operation. There are 2 ways to get this
  <br><br>  1. From directing to <code>http://local_pi_ip_address:5000</code><br>
  <strong>E.G</strong> <code>http://192.168.1.189:5000</code>
  <br>     2. Clicking 'Add Card'
  <br>     3. Clicking in the 'Card Code' Box
  <br>     4. Plug the RFID Reader into the computer you are on
  <br>     5. Scan the card
  <br>     6. Copy down the Code



<br><br>Or<br><br>    1. Connect the Pi to a Screen
  <br>     2. Open a terminal
  <br>     3. Plug in the RFID Reader
  <br>     4. Scan the card
  <br>     5. Copy down the code

<br><br><br><strong>    Action Name</strong> - The name of the action to call (Step 3)
<br> <br>  <strong> Link to album art (Optional)</strong> - Url to image of the album art
<br><br><strong>    Album Title (Optional) </strong>- The title of the album just made
<br><br><strong>    Artist (Optional)</strong> - The Artist of the Album
<br><br><strong>    ID</strong> - 9 digit unique number

<br><br><br>  <strong>6b. Through the GUI:</strong>
<br><br>  1.Visit the hosted server site in a browser: <code>http://local_pi_ip_address:5000</code>  <br>  <strong>E.G.</strong> <code>http://192.168.1.189:5000</code>
<br>  2. Click 'Add Card'
<br>  3. Either paste in an itunes link at the top (This has been temperamental for me) or fill in the details of your album.
<br>  4. Click the 'Card Code' Box
<br>  5. Scan the Card and the box should auto-fill
<br> 6. Save the card
<h5><li>Run the restart script (Just to be safe ygm):</li></h5>
1. Navigate to the magic-cards folder: <code>cd /home/pi/magic-cards</code><br><br>
2. Run the restart script: <code>script/restart</code>
<h5><li>Test it and pray to the gods of RFID</li></h5>
1. Scan the card you've configured for: <code>*boop*</code>
<h5><li>Repeat for all of them or think of some inventive method to massively improve my super janky technique</li></h5>
</ol>



<h2> Troubleshooting</h2>
My personal debug guide (idk if this is useful), work backwards. Look I even made a bloody flowchart for you god damn) (But all this is made from memory because I’m currently flying halfway across the world on a flight with no WiFi and I’m not allowed to use my laptop, so any issues. Ping me on discord (https://discord.gg/tK6n3hh) and I’ll help as much as I can from experience)<br><br>

Check the PDF I uploaded to refer yourself to the below steps.
<br><br>
Troubleshooting (1):
<br>There’s a problem with the card or the reader being detected.
<br>    * To check the reader connect the Pi to a screen, ensure the command line is open, tap the card on the reader, if you get an output then the reader works.
<br>    * Check the output in the command line is the same as that in the cards.json file.
  <br>  * If that is all correct, then check the config.json file has the correct even defined. Check step <ls -l devices step> to find the event number.
  <br>  * If you don’t get an output to the command line then the card reader isn’t connected or it is brok.
<br><br>
Troubleshooting (2):
<br>There’s a problem with magic-cards calling the script.
<br>    * Check actions.json is calling the correct .sh file
  <br>  * Check the .sh file is in the config folder
  <br>  * Check cards.json to ensure the card is calling the correct action.
<br><br>
Troubleshooting (3):
<br>There’s a problem with running the script
<br>    * Ensure you’ve edited the .sh file with the chmod +x command
<br>    * Ensure it is a script file
<br><br>
<br>Troubleshooting (4):
<br>There’s a problem with the command in the script file
  <br>  * Ensure that it is using the correct http or https
  <br>  * Ensure you’ve got the port included in the script command.
<br><br>
Troubleshooting (5):
<br>There’s a problem with the indexing and structure of your web server.
<br>    * Ensure you’re directing yourself to the same folder as follows /var/www/html/
  <br>  * Ensure the .mp3 is in the correct folder
<br><br>
<br>Bonus Error: ‘Can’t connect to the device’ error.
<br>Make sure that you can discover the device and the name is the same as in the ‘cast discover’ output.
