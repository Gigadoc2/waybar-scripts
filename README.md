# Waybar custom module collection
This are some (currently, some == 1) scrips meant to be used as
[waybar custom modules](https://github.com/Alexays/Waybar/wiki/Module:-Custom).

These are primarily made for my specific usage and as such, they might not fit
your usage and the quality is varying. However, they should be easy to adapt
to your usage.

## battery.py
A dbus-based battery indicator that is meant to replace the built-in one.

The main differences to the
[default module](https://github.com/Alexays/Waybar/wiki/Module:-Battery)
are:

  * It does not poll, instead it uses dbus and glib to be notified of changes.
    That way, the indicator will update in real-time, and it might even save
    some power.
  * It supports removing and adding batteries, which is useful if you have
    multiple batteries which are not always connected (so, a Thinkpad).
    As a consequence, if you use the wrong battery name, it will not error,
    but just think of that "battery" as disconnected.
  * It will display the percentage in relation of designed maximum capacity,
    instead of the percentage related to the current maximum capacity
    (see _Designed maximum capacity vs. actual maximum capacity_).

### waybar configuration
Include the module in the waybar config like this:
```json
    "custom/bat0": {
        "exec": "/path/to/battery.py BAT0",
        "return-type": "json",
        "format": "{icon} {}",
        "format-icons": ["", "", "", ""],
        "tooltip": false
    },
```

As you might guess, the parameter to `battery.py` defines the battery to be
monitored. The default already is BAT0, which in most cases would be your only
battery. If you have multiple however, the second one will probably be BAT1 and
so on.

As mentioned above, the battery does not have to be present (or exist at all).
`battery.py` will quietly wait until it appears (which might be never).

**Note:** The `{percentage}` output (which is used for the icon-selection, but
can also be used in the format string) is actually scaled from 0% to 100%,
unlike the textual output. This is mainly my personal preference so that, while
the text will display how much of the original capacity I have left, the icon
will grow fuller in relation to how much I can actually charge the battery.

#### Styling
The script will set various CSS classes depending on the battery state as
reported by UPower:

  * `unknown` - Don't know when this occurs, it has not happened to me yet
  * `charging` - When the battery is charging (not the same as the AC being
  plugged in!)
  * `discharging` - When the battery is discharging
  * `empty` - Pray you never see this
  * `full` - The battery is full
  * `charge_pending` - I have never encountered this
  * `discharge_pending` - Same as above

Additionally, it will append classes (without removing the above state class)
when the battery falls below a certain level (configured at the start of the
file, you might want to change these):

  * `critial` - The battery is 15% or lower
  * `warning` - The battery is between 16% and 30%

You can style a certain state in waybars `style.css` like so:
```css
#custom-bat0.charging,
#custom-bat1.charging {
    color: #9BC89D;
}
```

If you want a certain style for a warning level, but have it only apply when the
battery is not being charged (otherwise you would be warned about a critically
low battery while you are already recharging it):
```css
#custom-bat0.warning:not(.charging):not(.full),
#custom-bat1.warning:not(.charging):not(.full) {
    color: #fabd2f;
}
```

### Designed maximum capacity vs. actual maximum capacity
The default waybar battery module (and a lot of other battery indicators) will
show the percentage in relation to the current maximum capacity as opposed to
the design maximum capacity. In other words, as your battery ages and it's
capacity decreases, this will not be reflected in the indicator, it will still
show _100%_ (a notable exception from this is i3status).

Some people (me) however, would like to see the percentage in relation to it's
theoretical maxium capacity, as designed. I believe that that way, you can
better judge when to stop discharging your battery and search for a power
outlet. Consider this example scenario (which is in no way related to any real
experience I had):

Through repeated use, improper storage or deep discharging it, your battery is
left with 16% of it's original capacity, powering your notebook for a few
minutes only. With a normal battery indicator, it would still show _100%_, and
then just jump in really big steps towards _0%_. You would only notice that your
battery is about to deep discharge when you look at the time remaining (and then
also only when the time remaining is calculated in a helpful way), or when you
keep looking at the battery indicator and notice how fast it drops. You might
very well continue to deep-discharge it, and wonder why it is empty so quickly.

Instead, if you use an indicator like this that displays the percentage in
relation to the design capacity, you can immediately see that the battery is
dangerously low, even when fully charged, and can react accordingly.

Also, it will show you how badly your battery is broken.
