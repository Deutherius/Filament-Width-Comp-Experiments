# Filament Width Compensation Experiments
I have recently aquired quite a lot of filament (abs+ from esun) for a reasonable price. Problem is that while this filament batch is mostly within the usual spec (+-0.05 mm), the diameter oscillates min-to-max roughly every 10 cm. Which manifests on the print like this

![20220108_003617](https://user-images.githubusercontent.com/61467766/158036575-9ababa3b-92ce-41fe-b9b6-5acdf8d67a63.jpg)
![20220312_225950](https://user-images.githubusercontent.com/61467766/158036554-4453641a-8418-421b-87d9-af3d4a91dc0a.jpg)

You can see the prominent "Z-banding". After figuring out that it was indeed caused by the filament diameter, I went out in search of filament width sensors to try out filament width compensation.

What I found was a bunch of articles on the theory, a lot of designs from a few years back... But absolutely no pictures of "before and after", or any concrete data on the accuracy/repeatability of the designs. Oh, and few comments saying that with modern filaments, it's not worth the effort because if you are within +-0.05 mm, you are fine. Go figure.

In the end I settled on [this design](https://www.prusaprinters.org/prints/57699), mostly because it looked fleshed out, I could get all the components easily, it was cheap and it was already [supported in Klipper](https://www.klipper3d.org/Hall_Filament_Width_Sensor.html#hall-filament-width-sensor). I am using the version with bearings contacting the filament. Total cost is below 3 €.

There is also Thomas Sanladerer's design, the [InFiDEL](https://www.prusaprinters.org/prints/57154-infiDANK). Requires a bit more hardware, but could probably be equally viable. 

## Mounting and tuning
I mounted the sensor on my Voron 2.4 at the beginning of the reverse bowden tube (at the filament spool). I wanted to mount it closer to the toolhead, but there are some challenges. Namely, the sensors are sensitive to temperature, and bending of the filament in the reverse bowden can change the readings by putting differing pressure on the lever arm at different positions of the toolhead. Best compromise would be to mount the sensor at the very last point before the filament enters the chamber.

Next up, tuning the raw sensor values. I used a 1.5 mm allen key and a 2 mm drill bit (precise size verified with a micrometer), as is recommended in the guide. Fortunately, when filament was inserted, the values were corresponding with my measurements with an error on the order of units of micrometers. Pretty impressive. I tried multiple types of filament, all worked similarly well. If you have problems with this step, maybe consider the pre-tension spring being too strong and squishing the filament - I had to shorten mine a bit with snips to get good results. Also think about how the filament beds coming from the spool - ideally you want to measure it "from the sides" where it does not curve, since curving filament can put additional force on the measuring arm and skew the results.

Then I had to tune the distance between the sensor and nozzle. I got a rough measurement by pushing filament all the way in, snipping it at the sensor, pulling it out and measuring the length. This came out to roughly 128 cm. Just to be sure, I disconnected my hotend, made a contrasting mark at the sensor and ordered the extruder to push filament until the marked piece came out. This came out to 128.5 cm from sensor to nozzle end, or roughly 126.5 cm from sensor to melt zone (Dragon HF).

To be sure, I wanted to try a few distances to see how sensitive the algorithm is to this parameter. Since there is no "tuning tower"-style procedure for fine tuning, I elected to brute-force the problem by printing a 3-wall, 0-infill, 80x80x20 mm cube (with a 1 mm brim for adhesion) for every 10 mm of the sensor-nozzle distance, starting at 128.5 cm and going lower with every iteration. Such a cube consumes roughly 2.5 meters of filament, so any compensation should start happening roughly in the middle of the print.

## Results
Shown below are 3 cubes, from the left:
1. Control. No compensation.
2. Compensation on from a printer reset (clean filament width data buffer). Actual compensation kicks in roughly in the middle of the print.
3. Compensation on, no reset (filament width data buffer full from the previous print).

![combined](https://user-images.githubusercontent.com/61467766/158037713-ae745871-b9af-4c21-8006-64adf1171f6a.jpg)

Keep in mind that this is the harshest possible lighting I could provide, direct overhead white LEDs. Same thing, slightly more favorable lighting

![20220312_195422](https://user-images.githubusercontent.com/61467766/158037774-55e325a7-fbdd-4a61-ab86-84714747be60.jpg)

These 3 were taken with a distance parameter of 124.5 cm, but really anything from around 123 to 128 cm was usable. Other tests with Prusament ASA gave me the best subjective result at 126.5 cm, which is the setting I ended up using globally. Since it is a distance between the sensor and the start of the melt zone, it makes sense to me.

### Some updates

Here is the print from the first image in this repo, plus the same thing with comp enabled

![20220318_203013_rot](https://user-images.githubusercontent.com/61467766/159078096-7565f3bf-4499-4a3f-aa4d-a0bedceb52fe.jpg)
![20220318_204637](https://user-images.githubusercontent.com/61467766/159075490-897aa232-eb80-42aa-82c8-d49129ce0338.jpg)

It's far from perfect, but I am happy with it

## Logging
Klipper's built-in compensation code allows logging the measured diameter. This is a bit clunky, as it just pastes the measurements into the console at the measured intervals (I use 1 mm, lowest possible distance), which then gets saved to klippy.log. From there, you can extract the data and plot the filament diameter...

![quality_esun](https://user-images.githubusercontent.com/61467766/158818829-593c586c-c4db-4a67-b108-44cafe6ccd74.JPG)

...yikes. This correlates well with the observed effects (wide band of overextrusion followed by a thin band of underextrusion). Notably, this is still within the "standard" tolerance of +-0.05 mm, so the filament technically passed the basic QC requirement. Varying the diameter this much over such short distances *does* produce visible artefacts on the print though, as demonstrated. And even a cheap 3€ mechanical filament sensor can be used to fix the issue.

Note - I would not completely trust the X axis. I made a few prints that required 5m of filament but only got ~2200 log entries (I use 1 mm measurement_interval). I have yet to determine whether the logging is time based, time-limited, or what else is going on, so just... Take the X axis with a grain of salt.

## Disclaimer
I have been using the sensor for a grand total of 3 days at this point. No idea if these results are reliable in day-to-day printing as of yet. I have no idea about what accuracy/precision/repeatability to expect from the sensor, although the resolution seems to be about 1 micron. With good and consistent filaments, it is entirely possible that using the compensation would introduce more errors into the print than would otherwise be there, so use at your own risk*. Still, it makes my stash of esun ABS+ printable, so I am happy with it so far.

Also be aware that you might have to retune your EM with this mod. Since this process messes with flow rate, the slicer-side EM might be a bit different than before, i.e. if your filament is really consistent, but has a nominal mean diameter of 1.72 mm, your prints will look great, but your tuned EM might be higher than usual. The filament width compensation, however, expects 1.75 mm, so it will try to overextrude when it sees 1.72 mm. As a result, your slicer-side EM would actually need to be a bit lower than expected in this hypothetical example to get good prints.

It would also be beneficial to have at least 2 sensors working in unison to get a better measurement of the filament dimensions. Klipper is not ready for it yet though - maybe someone can add that into the existing module?

Also, as the sensors work in analog, they are susceptible to electrical noise - a lowpass filter near the board input pins might be advisable. Or go with a digital solution in the first place.

## Charts! Data!
I'll try to keep an up-to-date chart with various filament width logs here. At this time in a single chart, but I think it will become unreadable very quickly. I'll figure something out, probably.

So far we have:
1. ESUN ABS+ Black, new batch
2. Prusament PLA Mystic Green
3. ESUN ABS+ Natural, new batch
4. ESUN ABS+ Fire Engine Red, old batch
5. Prusament ASA Galaxy Black

![log_alt_colors](https://user-images.githubusercontent.com/61467766/158896189-59f3040e-d270-4e85-9973-1f519b22af2a.png)

## Effects on good filament

So what if you only run good filament? Do you need a filament width sensor? This is a test piece made from Prusament Galaxy Black ASA - you can see in the chart above that this filament has good diameter tolerances and stability. What does the compensation do with such a filament? You be the judge:

![20220318_210720](https://user-images.githubusercontent.com/61467766/159076478-7d08dc91-3a74-46aa-9fec-78019e3430f0.jpg)
![20220318_201029](https://user-images.githubusercontent.com/61467766/159076545-25f2d11f-6f92-41a5-be7d-bbc84cf82970.jpg)

In my opinion, not much, really. I can see a small improvement from halfway up if I squint really hard, but that could easily be chalked up to crappy EM tuning on my part (which the compensation fixes). Don't expect miracles, after all, this is still a robotic hot glue gun. At least the compensation doesn't seem to degrade the quality, which is a nice peace of mind.
