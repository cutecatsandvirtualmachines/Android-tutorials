https://play.google.com/store/apps/details?id=com.global.dqvn&hl=en_US

# Tools needed:

https://github.com/AndnixSH/APKToolGUI/releases

https://notepad-plus-plus.org/downloads/

https://github.com/frida/frida/releases

https://platinmods.com/threads/tool-encryption-decryption-jsc-files-to-modded-games-based-cocos2djs.132148/ 

# Preparation steps:

APK Tool GUI does NOT support Unicode characters such as ể , therefore you will need to rename the downloaded apk to something like Funtap_1.8.00.apk, then feed it into APK Tool GUI.

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/7cd8a164-50cf-418b-95f9-05de3f9e23ca)

# Observations:

Whenever you see a folder named jsb-adapter it means the game is using cocos engine: https://www.cocos.com/en
This engine is very widespread in asian games, and in general in asian game development studios.

The engine supports both lua and js engines, in this case it's using the js engine with encrypted .jsc files (for lua it would be .luac)

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/1d672e88-2895-4e16-b5d2-6f396000dede)

By examining main.js, which in cocos is the first (dev controlled) file that ever gets executed by the js engine, we can see the assetManager trying to load plugins (https://docs.cocos.com/creator/manual/en/scripting/external-scripts.html) in order:
1. main
2. script
3. start game

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/cb1b8107-04b7-446d-a91a-ec2282059da4)

For cocos2djs usually the script plugin contains the main game logic in a massive block, while main.js is used as a loader, sometimes implementing bundle caches and bundle updates (which you will need to defeat otherwise the game will be using cached scripts instead of your modified ones, especially in case of updates)

### P.S.
Plugins are each in its own folder inside assets/assets. For instance assets/assets/script is the script plugin.

To defeat the updates you will need to get the new plugins downloaded into the games folder with root (/data/data/{package_name}/files) and it will be located somewhere in there usually. Then you can manually put the new plugins inside the extracted apk and recompile it. This will stop anything from breaking or updating.

Otherwise you can delete the update apply code and delete the micro-update every time the game starts:
```
    clearPchIfRequired();
    continueToCopyTempPch();
    window.restoreCustomSearchPath();
```

in assets/main.js, replace it with:
```
    //clearPchIfRequired();
    //continueToCopyTempPch();
    //window.restoreCustomSearchPath();
    jsb.fileUtils.removeDirectory(window.pchStoragePath);
    jsb.fileUtils.removeDirectory(window.pchAddStoragePath);
    jsb.fileUtils.removeDirectory(window.pchStoragePathTemp);
    jsb.fileUtils.removeDirectory(window.pkgStoragePath);
```

You can also disable the update altogether but I didn't bother since microupdates are a couple kilobytes each.


Next we'll go into the js plugin and find out the js file is encrypted

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/a89b46e1-815c-4a2b-883e-bf9949a92c68)

We need to load up frida, for this you will need a rooted device with a frida server running, or you need to install a frida gadget inside the apk. Both of these won't be covered here, perhaps in another tutorial.

This is the script we'll use to dump the xxtea key:

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/110111ba-7caa-49ec-aced-8037858d7e00)

```
const doit = () => {
    const base = Process.findModuleByName("libcocos2djs.so");
    if (!base) {
      setTimeout(doit, 10);
    } else {
        var xxtea_decrypt = Module.findExportByName("libcocos2djs.so", 'xxtea_decrypt')
        console.log(`Pointer ${xxtea_decrypt} from base ${base}`)
        
        Interceptor.attach(xxtea_decrypt, {
            onEnter: function (args) {
                console.log(parseInt(args[3]))
                var key = Memory.readCString(args[2], 16)
                console.log('xxtea_decrypt:' + "key = " + key + ", keyLen = " + args[3]);
            },
            onLeave: function (retval) {
                
            },
        });
    }
};
  
doit();
```

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/30de5639-9776-4d4a-b617-7d451c010b69)

Now that we have the key: b20ac5e4-20c7-47 we can trash frida, it won't be useful anymore now that we can decrypt .jsc files!
Now you can either use Mika jsc tool, or my modified script:

Open a cmd while having python 3.7+ installed and do
```
pip install xxtea-py
pip install cffi
```

once unpacked go to the Decrypt.py script and open a console, then do:
```
py Decrypt.py b20ac5e4-20c7-47 path\to\extracted\apk\assets\assets
```

https://cdn.discordapp.com/attachments/1230848925200551996/1230866240654934067/JSC_Enc_Dec.zip?ex=6636dac4&is=66358944&hm=d1d2193cf5437bbfd7eb297f1e6f78ea735a9c4c164c6abd5e3783436de723e1&

going back to the plugins folders you will see they're all well and decrypted:

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/6f75b27f-a908-427c-918f-0df0b9ceebe7)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/6959d278-6a08-4507-aad1-edcff0dc4632)

you can delete the .jsc file if you want to save space, won't be needed anymore.

Going into notepad++ I'd suggest installing the JSTool plugin:

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/2f3968e1-ba5c-458f-b547-ab92a8e7b616)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/f0e8e96e-2769-412d-860a-0e37c0e2f7e5)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/ba6e80f7-ab44-4c8d-a43e-b04064fbc165)

now you have a decrypted and formatted script to work with
This contains the entirety of the game code, so one tip is to press ALT+0 to close all brackets
and you'll see single modules inside the game:

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/a4105005-83ea-4efa-a946-ccc4b5252c03)

You'll need to do some tinkering yourself to find out what's possible to modify and what's just visual. My suggestion is to completely delete certain pieces of code you think do something you need to modify, and see if the game crashes or misbehaves. If it doesn't then it means something you did was wrong.

You can start by doing some guesswork, for instance ctrl+f and look for unlocklv, you will find some interestin stuff. Let's try to change a function!

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/aa445019-9a93-487e-a911-66133badd4ed)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/71eb810c-a3c9-4826-9aa9-11e31b535c37)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/e7805055-0ce8-4ee8-b074-d066ce1ed11c)

to apply the changes, and rebuild the apk to install, you first need to go into the plugin directory and modify the config.json. Change encrypted to false, and delete the .jsc file.

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/0a2f13e8-c4ab-468f-bed4-837c548b2cd9)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/0bda4f6c-2cbc-46f5-a100-986d6caf3475)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/af7836d8-adb0-4b7f-8247-d4dab4bfcd7d)

then recompile and install the apk

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/0cf86409-12bf-4f85-875a-ee59bb697147)

#### And now we can use the auto chop feature!

We can also disable ads completely by instantly notifying the server we watched the ad before it's served to us!

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/bea8b196-4290-4903-b750-0919cf5a59be)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/b30af3f5-5ecf-4787-89a5-57c5606c5ad3)

We can also instantly skip battles!

![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/a815eb57-f3fe-452d-9793-551637655281)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/591332c6-2aea-4f40-9171-675cf5c77391)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/63f224b1-22c2-4918-a757-1fb0a1215135)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/e0b781ba-de03-4b34-964a-b1ce5acf9f35)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/989eed5b-fe2f-43de-b629-86538404d419)
![image](https://github.com/cutecatsandvirtualmachines/Android-tutorials/assets/145232977/d1689c12-3ba8-42ab-9c60-15e16fa1f21f)

# Last note
Battles are rate limited server side, so you'll still need to wait some time!
Most of stuff in game is networked so speeding up and skipping won't always be viable options unless you want to be kicked/banned from the server.









