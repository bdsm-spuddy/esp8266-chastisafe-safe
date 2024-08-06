## Chastisafe software

This is a version of the software for my digital safe ([v2](https://bdsm.spuddy.org/writings/Safe_v2/) and [v3](https://bdsm.spuddy.org/writings/Safe_v3/))
designed to work uniquely with [chastisafe](https://chastisafe.com)

Follow the instructions on that page to build the hardware, but the software
here can be uploaded to the ESP8266 board instead.  Do read the software
instructions on that page, though, to understand how to do initial setup,
configuring for WiFi, etc.

Instead of entering a combination, you can pick a running lock, instead.

### Diffences between the normal software.

On the original software you just needed to enter a username/password
to protect the safe from having an outsider change things.

In this version there is one more things to configure.  This is a bit hard...

* API access token

The "lockee" chastisafe APIs require a cookie to be set in order to access
them.  This is the access token.  To get it you need to look at your browser
and extract the cookie from there.  In Chrome this is done from the
developer tools which can be accessed from the top-right "vertical dots"
menu in chrome, or potentially by pressing F12 or alt+ctrl+i.  In the
developer menu, select the "Application" tab (click on the ">>" menu drop
down) and then on the left under Storage / Cookies you will see an entry
for "chastisafe".  It will look like a GUID; e.g. 12345678-1234-1234-1234-123456789abc

This is value we need, so copy that and paste it into the access token field
and click "Set API Details".

How you find this on another browser is an exercise left to the reader!

Now this token is valid for up to 3 days... from when it was set in your
browser!  So you might want to go back to the "change safe chastisafe details"
and click on the "renew now" link.  This will tell the safe to try and
renew the token.  You can see the new value on this screen.

The safe will also try and automatically renew this token every 12 hours.
This should allow for some errors; it'll try again in 12 hours time.

```diff
- WARNING: ACCESS TOKEN EXPIRATION
```

If, for some reason, the access token expires then the safe will not be able
to talk to the API.  You will need to manually get your cookie again.

(Reasons might be that you unplugged the safe from the power, or the network
broke, or...).

### Typical usage

* Setup the username/password and access token.

* Create the lock in the app

Create the lock as normal, but you don't need to look at the combination;
just go through the next screen.  You'll still get the random numbers to
help you forget, but if you didn't even look at the combination then
you'll not need to forget it!

* On the safe menu click on "Pick a lock".

If you have no locks running then you should see "No locks available
to pick".  Otherwise you will be given a list of locks (name, when it
was started, and keyholder); select the lock you want to load, and click
on Start lock.

If you get a 403 error here, it means the API access token is wrong (or
has expired).

The safe is now locked and you won't be able to open it until the lock
finishes on chastisafe.

### Refresh status

Every time you reload the web page or click the "Status" button, the
safe will reconnect to the API and check the lock status.  This isn't
always fast because the ESP8266 isn't a fast CPU.  It can take between
2 and 10 seconds, if not longer.  This is because we use TLS to prevent
"man in the middle" attacks.  After all, we don't want to be able to
unlock the safe early!

If there are communication issues then these will be displayed.

*IMPORTANT* if you lose internet access or the server goes down or
there's some other problem then you can't refresh the status!
ALWAYS ensure you have a backup key...

### Lock completion

Once a lock has reached "unlocked" state next time you click on the "status"
button then then the safe will delete the, which will then let you open it.

```
The lock has competed, and has been removed from the safe

The safe may be opened.
```

```diff
- WARNING: RECENT LOCKS
```

The chastisafe API only reports on the last 10 locks completed (whether
real or fake or abandoned).  After that the information is no longer
present.  You should make sure that you check the status (and so remove
the lock from the safe) of a completed lock as soon as possible, while
it is in the recent list.  Otherwise the safe won't know it's OK to be
opened, and will remain locked!

```diff
- WARNING: FAKE LOCKS
```
The chastisafe API doesn not expose any information about fake locks.  If
you run a lock with 5 fake copies then you will see 6 locks in the list.

e.g.
```
These locks are available:
o Test timer, started at 2024-08-06T19:11:36Z
o Test timer, started at 2024-08-06T19:11:36Z
o Test timer, started at 2024-08-06T19:11:36Z
o Test timer, started at 2024-08-06T19:11:36Z
o Test timer, started at 2024-08-06T19:11:36Z
o Test timer, started at 2024-08-06T19:11:36Z
```
For a self lock it probably doesn't matter which one you pick; you will play
all the locks on the chastisafe site as normal, but when you unlock on the
site you will click "status" on the safe to see if the safe will open.  Once
the safe is open you can abandon the other locks.

*HOWEVER* things get more complicated if there is a key holder.  A key holder
can see if a lock is fake or not, and you might (probably will have!) selected
a fake lock for the safe.  What the keyholder thinks is a fake lock is likely
to be the _real_ lock.  This could get very confusing so I don't recommend
using fake locks with keyholders, or else discuss it with them before hand
to be sure everyone is aware of this issues.

```diff
- WARNING: ABANDONED LOCKS
```

If you abandon a lock then the safe _will not open_.  This is to prevent
cheating.

## Deploying the software

The code repo contains a prebuilt `bin` files, built with 4M1M.  On
first startup this version of the software will write out the cert
files; there is no need to create or upload filesystem images 

On unix this can be written, from the command line with an instruction
similar to
```
python3 ~/.arduino15/packages/esp8266/hardware/esp8266/*/tools/esptool/esptool.py --port=/dev/ttyUSB0 write_flash 0x0 esp8266-chaster-safe.ino.bin
```

Windows also has an equivalent `esptool.exe` command.

Or you can use the `arduino-cli` command.  There is a Unix makefile included
to make this easy (`make upload`).

If rebuilding from the IDE then ensure you have set the Crystal speed to 160 rather than the default, just
to eek out that little extra performance (SSL is slow on these devices!).
I also recommend a 4M1M model.

## Programming the HW-622

I'm putting this here because that's what I use in the v3 hardware build.

This board comes in a couple of versions; the biggest difference is
whether headers at the top left are present. I recommend getting them
pre-installed because these boards don't have USB connectivity; you
program them using a 3.3V serial adapter connected to them.

Near the power supply input there are 3 pins

```
|   |   | G .
|   |   | R .  .  B
| + | - | T .  .  B
+---+---+
```

The G pin is Ground
The R pin is Rx
The T pin is Tx

You need a 3.3V USB-serial adapter.   Remember that Tx on your adaper may
need to be connected to Rx on the HW622 and vice versa.

Now to get the machine into program mode, you need to join the B pins and then
power on the board.  This _should_ be enough to get let `esptool` send
flash updates.

After programming has completed, remove the B link and then power cycle.

Note that you can also short the ESP reset pin to ground instead of
power cycling

```
                     +-------------+
  first pin here --> |             |
                     |             |
                     |             |
                     |             |
                     |             |
                     |             | <-- first pin here
                     +-------------+

                  |   |   |   .
                  |   |   |   .  .
                  | + | - |   .  .
                  +---+---+
```

## Initial configuration

You may want to do initial configuration and verification
of the software because the serial console displays lots of useful debug
output, and it's easy to get to this before you install into the safeo

e.g.

```
Starting...
Opening filesystem
Creating cert store
Writing
To write:191944
Complete
Total size of FS is: 957314
Used size of FS is: 193772
File list:
/certs.ar
Finished
Read length: 191944
Loading cert store
Number of CA certs read: 171
Getting passwords from EEPROM
Found in EEPROM:
  UI Username >>>AAA<<<
  UI Password >>>BBB<<<
  Wifi SSID   >>>CCC<<<
  Wifi Pswd   >>>DDD<<<
  Safe Name   >>>EEE<<<
  Dev Token   >>>FFF<<<
  API URL     >>>https://chastisafe.com/api-v1/<<<
  LockID      >>><<<
  Relay Pin   >>>4<<<

MAC: AA:BB:CC:DD:EE:FF
Connecting to CCC ...
1 2 3 4

Connection established!
IP address:     192.168.0.123
Hostname:       ESP-DD:EE:FF
Waiting for NTP time sync: ..
Current time (UTC): Mon Jul 29 00:02:52 2024

mDNS responder started
TCP server started
OTA service configured
```

# DISCLAIMER

If this code breaks for any reason and your safe can't be opened, then
I will not be held liable.  This code is provided with no warranty
whatsoever.  Always have an emergency escape process
