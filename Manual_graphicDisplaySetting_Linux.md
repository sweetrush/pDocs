### Manually Configure your Graphic Display 
- Using xrandr 
- Run this commands 
``` sh
 xrandr  # This will show the displays 

```

##### Adding a resolution 
``` sh
cvt [width  hight]

e.g: cvt 1920 1280

# Out of the command 
# 1920x1280 59.96 Hz (CVT) hsync: 79.57 kHz; pclk: 206.25 MHz
Modeline "1920x1280_60.00"  206.25  1920 2056 2256 2592  1280 1283 1293 1327 -hsync +vsync


```

##### Adding the Resolutions to the screen 

``` sh
xrandr --newmode "1920x1280_60.00"  206.25  1920 2056 2256 2592  1280 1283 1293 1327 -hsync +vsync  
```

##### Adding the new modes to be active on Displaypot-2
``` sh
xrandr --addmode [TheOutputDeviceDisplay] "The Displaymode"

xrandr --addmode DisplayPort-2 "1920x1080_60.00"

```

##### Activating the New Resolution 

``` sh
xrandr --output DisplayPort-2 --mode 1920x1080_60.00
```