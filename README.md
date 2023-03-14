# How to adapt a keyboard to use Peg
During Spring break 2023, I spent an afternoon porting [my keyboard](https://github.com/Maker-Gitsune/The-Splaynck) to use [Peg](https://peg.software/). The process was not that hard, but I felt I could have been faster since I thought the existing documentation was a bit sparse and was trying to apply what I saw in the github repo’s prebuilt code to my current code. The process is pretty straightforward; If you have a keyboard setup to use KMK, all that is needed is: a .json file which describes the physical key layout and optionally, and a boot.py, which disables the drive function except for when a specific key is held upon plugging in. After downloading Peg, here are the programs in the order I did them:

# kb.py
The easiest part. I was able to use my current one without modification. If it does not work, try comparing it with one for a similar keyboard from the [Peg github](https://github.com/boardsource/pegBoards) and going from there.

# boot.py
Also quite easy. I set this up some time ago; the one I am using is pretty much the demo one from the [KMK github](https://github.com/KMKfw/kmk_firmware/blob/master/docs/en/boot.md).

If you don’t currently have one, the peg github says that the layout.json can be configured to generate a boot.py, though I did not try it.

# main.py
This one was more involved. After several unsuccesful trial and errors, I based my main.py off the provided code for the [Boardsource 4x12](https://github.com/boardsource/pegBoards/tree/main/keyboards/Boardsource-4x12-blok). I took the module/extension handling bits in my main.py and replaced it with the stuff shown below.

    from kb import KMKKeyboard
    from kmk.keys import KC
    from kmk.modules.layers import Layers
    from kmk.modules.modtap import ModTap
    from kmk.hid import HIDModes
    from kmk.handlers.sequences import send_string
    import supervisor
    keyboard = KMKKeyboard()
    modtap = ModTap()
    layers_ext = Layers()
    keyboard.modules.append(layers_ext)
    keyboard.modules.append(modtap)
    keyboard.modules = [layers_ext, modtap]

Next, I changed the last two lines from:

    if __name__ == '__main__':
        keyboard.go()

to

    if __name__ == '__main__': 
        keyboard.go(hid_type=HIDModes.USB)

After that, I essentially nuked everything except for the keyboard.keymap section. The official documentation says that Peg is unable to process substituted key names so all of them had to go. I then setup 8 empty layers under keyboard.keymap as instructed in the official documentation:

    keyboard.keymap = [ [], 
    [], 
    [], 
    [], 
    [], 
    [], 
    [], 
    [] ]

The last part is to setup a default/first layer under keyboard.keymap. Again, Peg is unable to process custom keycodes so the layer must be a minimum working layer for your keyboard that uses [**stock KMK keycodes**.](https://github.com/KMKfw/kmk_firmware/blob/master/docs/en/keycodes.md) Mine looked like:

    # keymap
    keyboard.keymap = [ [KC.ESCAPE,KC.Y,KC.P,KC.O,KC.U,KC.J,KC.K,KC.D,KC.L,KC.C,KC.W,KC.ENTER,KC.TAB,KC.I,KC.N,KC.E,KC.A,KC.COMMA,KC.M,KC.H,KC.T,KC.S,KC.R,KC.TAB,KC.CAPSLOCK,KC.Q,KC.Z,KC.SLASH,KC.DOT,KC.SCOLON,KC.F5,KC.B,KC.F,KC.G,KC.V,KC.X,KC.NO,KC.LGUI,KC.LALT,KC.DELETE,KC.RSHIFT,KC.SPC,KC.BSPC,KC.RALT,KC.RGUI], 
    [], 
    [], 
    [], 
    [], 
    [], 
    [], 
    [] ] 
    # keymap

**Important:** you need to comment “keymap” as follows before and after keyboard.keymap so that peg can find and edit it.

Here is what my finished main.py looked like:

    from kb import KMKKeyboard
    from kmk.keys import KC
    from kmk.modules.layers import Layers
    from kmk.modules.modtap import ModTap
    from kmk.hid import HIDModes
    from kmk.handlers.sequences import send_string
    import supervisor
    keyboard = KMKKeyboard()
    modtap = ModTap()
    layers_ext = Layers()
    keyboard.modules.append(layers_ext)
    keyboard.modules.append(modtap)
    keyboard.modules = [layers_ext, modtap]
    
    # keymap
    keyboard.keymap = [ [KC.ESCAPE,KC.Y,KC.P,KC.O,KC.U,KC.J,KC.K,KC.D,KC.L,KC.C,KC.W,KC.ENTER,KC.TAB,KC.I,KC.N,KC.E,KC.A,KC.COMMA,KC.M,KC.H,KC.T,KC.S,KC.R,KC.TAB,KC.CAPSLOCK,KC.Q,KC.Z,KC.SLASH,KC.DOT,KC.SCOLON,KC.F5,KC.B,KC.F,KC.G,KC.V,KC.X,KC.NO,KC.LGUI,KC.LALT,KC.DELETE,KC.RSHIFT,KC.SPC,KC.BSPC,KC.RALT,KC.RGUI,KC.NO], 
    [], 
    [], 
    [], 
    [], 
    [], 
    [], 
    [] ] 
    # keymap
    if __name__ == '__main__': 
        keyboard.go(hid_type=HIDModes.USB)

# layout.json
This is the last part. I once again started with the one for the [Boardsource 4x12](https://github.com/boardsource/pegBoards/tree/main/keyboards/Boardsource-4x12-blok) after some unsuccesful attempts. In a text editor, I first edited the features heading to reflect my keyboard; “bootSize” was set to 0 since I had a working boot.py and the “name” and “creator” fields were changed as follows:

    "features": {
            "perkey": false,
            "oled": false,
            "ble": false,
            "underglow": false,
            "name": "Splaynck",
            "creator": "Maker-Gitsune",
            "perkeyCount": 0,
            "underglowCount": 0,
            "split": false,
            "rightSide": false,
            "encoders": false,
            "encoderCount": 0,
            "rx_tx": false,
            "uartFlip": false,
            "splitPico": true,
            "bootSize": 0
        },

I went to [KLE](http://www.keyboard-layout-editor.com/) and constructed the layout of my keyboard. Do not use the export json option on KLE. The resulting json is not in the same order as the ones on the Peg github. Instead, once you have constructed the physical layout (without legends), go under the “raw data" tab and copy everything into [QMK JSON converter](https://qmk.fm/converter/) and click convert.

Replace the

    "layout": [
                ],

in the sample json with the one from [QMK JSON converter](https://qmk.fm/converter/). You need to add information to this, specifically the size of the keys (I’m not sure if it is possible to specify keys taller than 1u). To do this, add to each set of brackets; it must be in the format {“w”: “x”: “y”:} as shown below:

    "layout": [{"w":1.5, "x":0, "y":0},
               {"w": 1, "x":1.5, "y":0}, 
               {"w": 1, "x":2.5, "y":0}, 
               {"w": 1, "x":3.5, "y":0}, 
               {"w": 1, "x":4.5, "y":0}, 
               {"w": 1, "x":5.5, "y":0}, 
               {"w": 1, "x":6.5, "y":0}, 
               {"w": 1, "x":7.5, "y":0}, 
               {"w": 1, "x":8.5, "y":0}, 
               {"w": 1, "x":9.5, "y":0}, 
               {"w": 1, "x":10.5, "y":0}, 
               {"w": 1.5, "x":11.5, "y":0}, 
               {"w":1.25, "x":0, "y":1}, 
               {"w": 1, "x":1.25, "y":1}, 
               {"w": 1, "x":2.25, "y":1}, 
               {"w": 1, "x":3.25, "y":1}, 
               {"w": 1, "x":4.25, "y":1}, 
               {"w": 1, "x":5.25, "y":1}, 
               {"w": 1, "x":6.75, "y":1}, 
               {"w": 1, "x":7.75, "y":1}, 
               {"w": 1, "x":8.75, "y":1}, 
               {"w": 1, "x":9.75, "y":1}, 
               {"w": 1, "x":10.75, "y":1}, 
               {"w":1.25, "x":11.75, "y":1}, 
               {"w": 1, "x":0, "y":2}, 
               {"w": 1, "x":1, "y":2}, 
               {"w": 1, "x":2, "y":2}, 
               {"w": 1, "x":3, "y":2}, 
               {"w": 1, "x":4, "y":2}, 
               {"w": 1, "x":5, "y":2}, 
               {"w": 1, "x":6, "y":2}, 
               {"w": 1, "x":7, "y":2}, 
               {"w": 1, "x":8, "y":2}, 
               {"w": 1, "x":9, "y":2}, 
               {"w": 1, "x":10, "y":2},
               {"w": 1, "x":11, "y":2}, 
               {"w": 1, "x":12, "y":2}, 
               {"w": 1, "x":2.5, "y":3}, 
               {"w": 1, "x":3.5, "y":3}, 
               {"w": 1, "x":4.5, "y":3}, 
               {"w": 1, "x":5.5, "y":3}, 
               {"w": 1, "x":6.5, "y":3}, 
               {"w": 1, "x":7.5, "y":3}, 
               {"w": 1, "x":8.5, "y":3}, 
               {"w": 1, "x":9.5, "y":3}
            ],
#  Finishing up
Once this is finished, load your boot.py, main.py, kb.py and layout.json onto your keyboard and launch Peg. After it detects your keyboard, you should be greeted by your default layer mapped to your physical layout. From here, you need to enable dev mode under settings to be able to add custom keys.  You should now be able to drag and drop letters etc. onto the virtual keyboard and click “save map” to save the mapping to your keyboard.

At this point, I tried adding some custom keys and they mostly worked; if it was supported by the modules and extensions in the main.py, it was accepted without issue, but if I tried adding a module/extension to the main.py, saving and then trying to add a custom key using it, the keyboard would stop working. Because of that, I still prefer to use a text editor to remap my keyboard.

The code in the examples is [here](https://github.com/Maker-Gitsune/The-Splaynck/tree/main/Splaynck%20code/Peg%20stuff).
