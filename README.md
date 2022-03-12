# Filament Width Compensation Experiments
I have recently aquired quite a lot of filament (abs+ from esun) for a reasonable price. Problem is that while this filament batch is mostly within the usual spec (+-0.05 mm), the diameter oscillates min-to-max roughly every 10 cm. Which manifests on the print like this

![20220108_003617](https://user-images.githubusercontent.com/61467766/158036575-9ababa3b-92ce-41fe-b9b6-5acdf8d67a63.jpg)
![20220312_225950](https://user-images.githubusercontent.com/61467766/158036554-4453641a-8418-421b-87d9-af3d4a91dc0a.jpg)

You can see the prominent "Z-banding". After figuring out that it was indeed caused by the filament diameter, I went out in search of filament width sensors to try out filament width compensation.

What I found was a bunch of articles on the theory, a lot of designs from a few years back... But absolutely no pictures of "before and after", or any concrete data on the accuracy/repeatability of the designs. Oh, and few comments saying that with modern filaments, it's not worth the effort because if you are within +-0.05 mm, you are fine. Go figure.

In the end I settled on [this design](https://www.prusaprinters.org/prints/57699), mostly because it looked fleshed out, I could get all the components easily and it was already supported in Klipper. I am using the version with bearings contacting the filament.

## Mounting and tuning
I mounted the sensor on my Voron 2.4 at the beginning of the reverse bowden tube.

I wanted to mount it closer to the toolhead, but there are some challenges. Namely, the sensors are sensitive to temperature, and bending of the filament in the reverse bowden can change the readings by putting differing pressure on the lever arm at different positions of the toolhead. Best compromise would be to mount the sensor at the very last point before the filament enters the chamber.

Then I had to tune the distance between the sensor and nozzle. I got a rough measurement by pushing filament all the way in, snipping it at the sensor, pulling it out and measuring the length. This came out to roughly 128 cm. I would not trust this measurement.

Since there is no "tuning tower"-style procedure for fine tuning this distance, I elected to brute-force my way to the correct value by printing a 3-wall, 0-infill, 80x80x20 mm cube (with a 1 mm brim for adhesion) for every 10 mm of the sensor-nozzle distance, starting at 128 cm and going lower with every iteration. Such a cube consumes roughly 2.5 meters of filament, so any compensation should start happening roughly in the middle of the print.

## Results
Shown below are 3 cubes, from the left:
1. Control. No compensation.
2. Compensation on from a printer reset (clean filament width data buffer). Actual compensation kicks in roughly in the middle of the print.
3. Compensation on, no reset (filament width data buffer full from the previous print).

![combined](https://user-images.githubusercontent.com/61467766/158037713-ae745871-b9af-4c21-8006-64adf1171f6a.jpg)

Same thing, slightly more favorable lighting

![20220312_195422](https://user-images.githubusercontent.com/61467766/158037774-55e325a7-fbdd-4a61-ab86-84714747be60.jpg)

