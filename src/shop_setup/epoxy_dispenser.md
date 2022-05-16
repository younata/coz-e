# Automated Epoxy Dispenser

When you use a traditional epoxy pump, there's no way to tell if it actually dispensed in the correct ratio until the epoxy has cured. If there's some kind of blockage in one of the pumps, you won't find out until the epoxy has cured, and you find out it didn't cure correctly. Because of this, you need to calibrate the pumps regularly.

Still, fear of this is why I elected to use a scale and manually pour epoxy using a scale to measure it. However, manually pouring epoxy is a bit of a skill. Not a hard skill to learn - I had basically mastered it by the fourth or fifth time I mixed epoxy. But it's a bit annoying and time consuming. Certainly, pulling levers for epoxy pumps is significantly easier to do, and it takes significantly less time to do so. I wanted something even easier, even if it's still not as fast. As a wannabe electrical engineer, I decided to build an automated epoxy dispenser.

So, I bought some peristaltic pumps and some other components, and I made the simplest "press button, receive epoxy" program I could think of.

## Current State

![Image of the Epoxy Dispenser in place, with the pumps on top of the epoxy box, the control electronics on the right, and the scale towards the bottom](/assets/images/epoxy_dispenser/epoxy_setup.jpg)

The epoxy dispenser I built works very well. With the press of a button (the green button on the box on the right), it starts dispensing. Currently, it's set to dispense 120 grams (about 4 oz) total of epoxy and hardener. While it does take quite some time to dispense (approximately 10 minutes), it's worth it to not have to manually pour epoxy. This would be an issue if I used epoxy with a faster working time, but the MGS 335 system I use has a working time measured in hours.

### Hardware

This uses 2 peristaltic pumps to dispense epoxy. [Wikipedia has a pretty good article about them](https://en.wikipedia.org/wiki/Peristaltic_pump), but peristaltic pumps work by squeezing a plastic tube to force fluid through it. This makes them great for a ton of applications where you want to pump something, but you don't want to expose the pump itself.

These pumps are controlled by an arduino clone, using an L293D motor driver chip. This is a common driver chip and it lets me to control a motor from a low-amperage-output control pin.

On the sensing side, I have a basic load cell hooked up to an HX711 amplifier chip.

The arduino clone reads the load cell (via the HX711 chip), and then uses that to determine which pump to run, as well as what speed to run it at.

### Program

The control program runs at a rate of 10 hz. Each loop, it samples the scale to get the current total weight, and runs that through a PID loop to determine how quickly to run the chosen pump. Additionally, it also updates the character display with information about the current state of the system, as well as how much of resin or hardener it has dispensed.

On a higher level, the program is a simple state machine: It boots up into the IDLE state, where does nothing but take measurements from the load cell and display them on the character display. This makes the program useful for manually pouring epoxy, on the occasions I need to do that. When you press the green button, it changes to the RESIN state, where it runs the resin pump. Once the controller has detected that enough resin has been dispensed, then it switches over to the HARDENER state, where it turns off the resin pump and runs the hardener pump. Once the controller detects that we've dispensed enough hardener, it switches to the SHUTDOWN state, where it turns off all pumps and resets back to the IDLE state.

### Issues

As much as I like this system, it's not without issues. On the software side, I don't have a way to pause dispensing without stopping and resetting. So if I run out of either resin or hardener mid-pour, then I need to either throw away what was already dispensed, or manually finish the pour. On a future revision, I may add a switch to allow me to pause without resetting.

![Epoxy Hardener crystalizing on the plastic tubing](/assets/images/epoxy_dispenser/hardener_osmosing_issue.jpg)

More worryingly than issues replacing the contains, the epoxy hardener I use appears to be able osmose through the plastic tubing of the pumps. Which makes me worry that it might lead to gumming up the pump, potentially breaking it or causing some other issue. This appears to be the same issue that [@cyanocracy on twitter](https://twitter.com/cyanoacry) ran into [when he tried this same idea](https://twitter.com/cyanoacry/status/1520564688840396800), though he's using a slightly different epoxy system. If I can't resolve or mitigate this issue, then I might have to go back to the metaphorical drawing board. Maybe I'll get one of the manual epoxy pumps, and automate that using linear actuators on the handles?
