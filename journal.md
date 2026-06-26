# Build Journal

A running devlog of progress on the wind tunnel build, oldest to newest.

---

## Devlog #1 — Desktop Wind Tunnel

Hi everyone! For those who don't know, a wind tunnel is a device used to study aerodynamics by passing a controlled airstream over test objects. My goal is to create a fully 3D-printed wind tunnel that's easy to assemble and doesn't require hardware like nuts and bolts.

So far, I've made a 3D-printed mist maker that sits at the inlet, allowing you to actually visualize the airflow as it moves through the tunnel. It looks SUPER cool when flowing around test objects like Hot Wheels cars!

The fan driving the airstream is a Noctua NF-R8 redux-1800, a 4-pin PWM fan that runs on 12V. PWM control allows precise adjustment of the airflow speed. To control the fan, I've built a dedicated electronics controller based on an ESP32, and I plan to integrate everything into the 3D-printed enclosure for a clean, sleek final design.

---

## Devlog #2 — Desktop Wind Tunnel

*17 days ago · 1h 11m 53s logged*

I printed and assembled the test chamber, which is where the test objects sit. I also had to cut acrylic/plexiglass windows to keep the test object visible while having the test chamber sealed for the airstream to flow properly.

---

## Devlog #3 — Desktop Wind Tunnel

*16 days ago · 2h 29m 12s logged*

V1 is done! Unfortunately, my lapse with the slicing, cleaning, and assembly got corrupted, but over these past 2 weeks, I was able to get all parts printed, assembled, and tested. The tunnel runs end to end with the mist maker producing visible flow and the knob controller driving the fan. Testing surfaced a few clear improvements for V2: the diffuser fan needs screw mounts (not secure enough right now), the test section could use a visual renovation (IB: 64 Windsible), and all the electronics need a proper enclosure rather than sitting loose.

---

## Devlog #4 — Desktop Wind Tunnel

*4 days ago · 10h 33m 9s logged*

After the completion of V1, I went in and polished the firmware to look more like the 64 Windsible, so rather than displaying 100%, it'll say 250 mph. This makes the tunnel more fun and gives the user an understanding of what speed the test object would be moving at. I updated the repository to have the CAD files and a better README. I also rewired the circuit to remove the breadboard entirely, making it easier to embed in V2. My next steps are to ship V1 in hopes of going to Outpost and start V2 sometime next week.

---

## Devlog #5 — Desktop Wind Tunnel

*3 days ago · 1h 13m 59s logged*

I didn't change much, but debugged an issue with the mph knob display update where the float value wasn't displaying on the knob properly.
