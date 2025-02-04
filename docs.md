# UV scheme for animating the pixel strips

- Texture
    - Tiling: The fraction of the full image to be displayed
        - X
            - naive: `1/128`
            - omitting leftmost pixel: `(7/8)/128`
            - addressing UV bleeding: 
                - `del_frac = 1/128`
                - `((7/8)/128)*(1-del_frac)`
        - Y
            - naive: `1/128`
            - better: `1e-13`
            - even better: `0.5/128`

    - Offset: Starting at the far left, moving right. Starting at the bottom, moving up.
        - X
            - naive: `color_int/128`
            - omitting leftmost pixel: `color_int/128 + 1/1024`
            - animation: from `0 + 1/1024` to `127/128 + 1/1024`
            - addressing UV bleeding: 
                - `color_int/128 + 1/1024`
                - `+ ((7/8)/128)*(del_frac/2)`
            - animation:
                - from
                    - `0/128 + 1/1024`
                    - `+ ((7/8)/128)*(del_frac/2)`
                - to
                    - `127/128 + 1/1024`
                    - `+ ((7/8)/128)*(del_frac/2)`, putting the entire constant offset in `PxColors`
                      - apparently constant offsets are ignored in additive animations??????????
        - Y
            - manually in editor, naive: `px_int/128`
            - manually in editor, better: `(px_int + 0.5)/128`
            - animation: from `0.5/128` to `127.5/128`
            - manually in editor, even better: `(px_int + 0.25)/128`
            - animation: from `0/128` to `127/128`, putting a constant `0.25/128` offset in `PxColors`
                - apparently constant offsets are ignored in additive animations??????????








# Process for creating the Unity portion

Create an empty PxScreen object. This can be moved and scaled later.

(Create all pixels 1m wide initially, so that position math in the inspector is easy.)

Each pixel strip is a quad, 7m by 1m. They're called PxStripL1 through PxStripL8 on the left, and PxStripR1 through PxStripR8 on the right.


## Making pixels light or dark, with UV scrolling


### Materials

They should each get their own material, using pixel-strip-color.png, using the same PxStrip names. (For testing without adding color, start with the monochromatic `pixel-strip.png`.)


### Params

In the avatar parameters, they should each get their own synced float params, using the same PxStrip names. Only check "Saved" if you want to retain the image between sessions. Personally, I prefer to default to a smiley face, until OSC is enabled.


### Animations

They should each get their own animations, with the same PxStrip names.

To create an animation, create a copy of the avatar, because the Unity animation "record" or "preview" feature might get the avatar stuck in the motorcycle pose.

To create an animation on an asset that'll be *distributed*, package the PxScreen as a prefab, and animate relative to the root of the prefab, rather than relative to the root of the avatar.

Create an animation with two frames - 0 and 1, or 0 and 127. Record and scroll UVs with the Inspector, according to the math described in the first part of this file. After recording, toggle Preview off.

Automate this with a tool like TinyTask.


### Params in the FX controller

They should each get their own FX controller float param, with the same PxStrip names.


### Animation Layers

They should each get their own animation layers, with the same PxStrip names:
- Weight: 1
- **Additive**, not Override. (If prototyping in black and white, use Override.)
- Motion Time: corresponding PxStrip


## Picking a color palette, with UV scrolling

(If you prototyped in black and white, then switch each Override layer to Additive, and switch each material to `pixel-strip-color.png`.)


### Materials

You don't need a new material.

### Params

Add a PxColors synced float param. I like to Save it, to keep the color from last time.

### Animations

Create one PxColors animation. Animate every pixel strip, according to the math described in the first part of this file.

Automate this with a tool like TinyTask.

### Param in the FX controller

Add a float called PxColors.

### Animation Layer

Create an animation layer that goes *above* all the PxStrip layers:
- Weight: 1
- Override
- Motion Time: PxColors







# Palette scheme

There are 16 light "foreground" colors, and 8 dark "background" colors. They mostly overlap, for a total of 17 colors. But the foreground removes black, and adds orange and sky blue.

Currently, the palette image is split up into 8 total regions representing the background color, and each region is subdivided into 16 sub-regions for each foreground color.

(This may be reversed later - 16 large foreground regions, and 8 smaller background sub-regions - so that PxColors can be used to animate a flashlight color, drawn from the light foreground color.)

If the foreground and background are the same color, then the foreground is manually lightened in `pixel-strip-color.png`.


## Alternate scheme for more palettes

Instead of 7-pixel strips with 1 unused bit, we could switch to 2x3-pixel blocks with 2 unused bits.

We could then use the 2 bits to pick an alternate color scheme.

### Alternate scheme 1: More brightness control

One bit can set the background to a darker color (usually black), and one bit can set the foreground to a lighter color (usually white).
- Surprisingly, this seems more promising, mostly because it's often useful to replace the background with black.
- We can make red go to orange or yellow, and orange go to yellow.
- We can make dark colors go to their lighter version.

### Alternate scheme 2: More hue control

The 2 bits can be used to select any of 3 alternate palettes, which preserve lightness but provide alternative hues, probably for the foreground color.


## Alternate scheme for more color depth

Instead of 7-pixel strips, we can use 3-pixel strips, using 6 bits. (We can reuse the extra 2 bits as described above.)

The 3-pixel strips use 2-bit monochrome.

We make up for the reduction in pixels by adding 