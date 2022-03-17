# Filament Width Compensation Experiments
I have recently aquired quite a lot of filament (abs+ from esun) for a reasonable price. Problem is that while this filament batch is mostly within the usual spec (+-0.05 mm), the diameter oscillates min-to-max roughly every 10 cm. Which manifests on the print like this

![20220108_003617](https://user-images.githubusercontent.com/61467766/158036575-9ababa3b-92ce-41fe-b9b6-5acdf8d67a63.jpg)
![20220312_225950](https://user-images.githubusercontent.com/61467766/158036554-4453641a-8418-421b-87d9-af3d4a91dc0a.jpg)

You can see the prominent "Z-banding". After figuring out that it was indeed caused by the filament diameter, I went out in search of filament width sensors to try out filament width compensation.

What I found was a bunch of articles on the theory, a lot of designs from a few years back... But absolutely no pictures of "before and after", or any concrete data on the accuracy/repeatability of the designs. Oh, and few comments saying that with modern filaments, it's not worth the effort because if you are within +-0.05 mm, you are fine. Go figure.

In the end I settled on [this design](https://www.prusaprinters.org/prints/57699), mostly because it looked fleshed out, I could get all the components easily and it was already supported in Klipper. I am using the version with bearings contacting the filament.

## Mounting and tuning
I mounted the sensor on my Voron 2.4 at the beginning of the reverse bowden tube (at the filament spool). I wanted to mount it closer to the toolhead, but there are some challenges. Namely, the sensors are sensitive to temperature, and bending of the filament in the reverse bowden can change the readings by putting differing pressure on the lever arm at different positions of the toolhead. Best compromise would be to mount the sensor at the very last point before the filament enters the chamber.

Then I had to tune the distance between the sensor and nozzle. I got a rough measurement by pushing filament all the way in, snipping it at the sensor, pulling it out and measuring the length. This came out to roughly 128 cm. Just to be sure, I disconnected my hotend, made a contrasting mark at the sensor and ordered the extruder to push filament until the marked piece came out. This came out to 128.5 cm from sensor to nozzle end, or roughly 126.5 cm from sensor to melt zone (Dragon HF).

To be sure, I wanted to try a few distances to see how sensitive the algorithm is to this parameter. Since there is no "tuning tower"-style procedure for fine tuning, I elected to brute-force the problem by printing a 3-wall, 0-infill, 80x80x20 mm cube (with a 1 mm brim for adhesion) for every 10 mm of the sensor-nozzle distance, starting at 128.5 cm and going lower with every iteration. Such a cube consumes roughly 2.5 meters of filament, so any compensation should start happening roughly in the middle of the print.

## Results
Shown below are 3 cubes, from the left:
1. Control. No compensation.
2. Compensation on from a printer reset (clean filament width data buffer). Actual compensation kicks in roughly in the middle of the print.
3. Compensation on, no reset (filament width data buffer full from the previous print).

![combined](https://user-images.githubusercontent.com/61467766/158037713-ae745871-b9af-4c21-8006-64adf1171f6a.jpg)

Same thing, slightly more favorable lighting

![20220312_195422](https://user-images.githubusercontent.com/61467766/158037774-55e325a7-fbdd-4a61-ab86-84714747be60.jpg)

These 3 were taken with a distance parameter of 124.5 cm, but really anything from around 123 to 128 cm was usable. Other tests with Prusament ASA gave me the best subjective result at 126.5 cm, which is the setting I ended up using globally. Since it is a distance between the sensor and the start of the melt zone, it makes sense to me.

## Logging
Klipper's built-in compensation code allows logging the measured diameter. This is a bit clunky, as it just pastes the measurements into the console at the measured intervals (I use 1 mm, lowest possible distance), which then gets saved to klippy.log. From there, you can extract the data and plot the filament diameter...

![quality_esun](https://user-images.githubusercontent.com/61467766/158818829-593c586c-c4db-4a67-b108-44cafe6ccd74.JPG)

...yikes. This correlates well with the observed effects (wide band of overextrusion followed by a thin band of underextrusion). Notably, this is still within the "standard" tolerance of +-0.05 mm, so the filament technically passed the basic QC requirement. Varying the diameter this much over such short distances *does* produce visible artefacts on the print though, as demonstrated. And even a cheap 3€ mechanical filament sensor can be used to fix the issue.

## Disclaimer
I have been using the sensor for a grand total of 3 days at this point. No idea if these results are reliable in day-to-day printing as of yet. With good and consistent filaments, it is entirely possible that using the compensation would introduce more errors into the print than would otherwise be there, so use at your own risk. Still, it makes my stash of esun ABS+ printable, so I am happy with it so far.

Also be aware that you might have to retun your EM with this mod. Since this process messes with flow rate, the slicer-side EM might be a bit different than before, i.e. if your filament is really consistent, but has a nominal mean diameter of 1.72 mm, your prints will look great, but your tuned EM might be higher than usual. The filament width compensation, however, expects 1.75 mm, so it will try to overextrude when it sees 1.72 mm. As a result, your slicer-side EM would actually need to be a bit lower than expected in this hypothetical example to get good prints.

It would also be beneficial to have at least 2 sensors working in unison to get a better measurement of the filament dimensions. Klipper is not ready for it yet though - maybe someone can add that into the existing module?

## Charts! Data!
I'll try to keep an up-to-date chart with various filament width logs here. At this time in a single chart, but I think it will become unreadable soon. I'll figure something out, probably.

So far we have:
1. ESUN ABS+ Black, new batch
2. Prusament PLA Mystic Green
3. ESUN ABS+ Natural, new batch
4. ESUN ABS+ Fire Engine Red, old batch

![log_alt_colors](https://user-images.githubusercontent.com/61467766/158882988-a012d516-23ac-4c5b-8467-17270c5400e1.JPG)

