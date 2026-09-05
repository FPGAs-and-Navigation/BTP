# Speaker Notes: MidTerm Evaluation

**Total: 6:00 to present + 1:00 for questions (4 presenters, split however you like).**
Slides only carry headlines and diagrams: use the notes below to fill in the actual sentences out loud.
References (slides 8-9) are backup only: don't present them, just have them ready if someone asks "what's your source for X".

---

### Slide 1: Title (~0:10)
Quick intro: project name, that this is the MidTerm evaluation, and the four names on the slide.

### Slide 2: Objectives (~0:40)
- We want a UAV to know its own position without GPS: jammed, spoofed, or just unavailable, using only what it already carries onboard.
- This semester's bar: a pipeline that works end-to-end in simulation, image/data in, coordinates out.
- Next semester: make that pipeline fast and power-efficient enough to actually fly on embedded hardware (FPGA).
- The core idea, "Scene-Aided Navigation," is to replace the GPS signal with things the aircraft can already physically sense (terrain, gravity, magnetic field, camera view) and match those against a known reference map. Same philosophy as classical inertial guidance, just with a richer set of sensors.

### Slide 3: Motivation (~0:55)
- GNSS is a very weak signal broadcast from space, trivial to jam or spoof, and it simply doesn't reach some places (urban canyons, mountains, contested airspace, indoors).
- That matters because the UAVs that most need GPS-denied capability, namely survey, disaster response, and defense platforms, are exactly the ones that must keep flying when GPS is taken away.
- Mechanical framing: a UAV is a mechanical system in motion, and its IMU is already measuring acceleration, rotation, and gravity. The question we're asking is whether those existing measurements, plus a camera, can substitute for the satellite link entirely.
- This isn't a new idea. TERCOM and DSMAC did exactly this (terrain/scene matching) for Cold-War cruise missiles, and it's flight-proven.

### Slide 4: Background of Similar Work (~1:10)
- Walk the table left to right, don't read every row verbatim, group them instead:
  - Visual/terrain matching (TERCOM, visual landmark, transformer-based): all passive, all camera- or elevation-based; this is the strongest cluster and where most of the recent literature sits.
  - Gravity and gravity-gradient: passive, promising, but needs a sensitive sensor; the higher-precision variant needs even more sensitivity.
  - Magnetic anomaly: useful specifically in mountainous terrain where the field varies enough to be distinctive.
  - Quantum inertial sensing: no map needed at all, pure mechanics; flagged as an interesting idea to revisit later.
  - Wi-Fi/cellular/RF: rejected outright, doesn't work at altitude.
- Call out the "Transformer-based matching" row specifically: this is newer (TransGeo/FSRA-style work) and state-of-the-art in this specific niche. It's what's motivating our redesign, which the next two slides cover.
- Last line: implementations we actually reviewed are VGG16 and CLIP-based deep features, classical SIFT/ORB, and DEM terrain-weighted optimization, landing around 3-7 m RMSE in the literature.

### Slide 5: Proposed Work (~1:00)
- Point at the diagram: right now, no drone hardware, so we feed pre-recorded aerial image/video plus gravimetric/magnetic data into the localization model and get lat/long out.
- Baseline validated: we replicated the CMU reference paper (VGG16 features plus inverse-compositional alignment) ourselves in simulation, and our numbers match what the paper reports. That's done; it's not a proposal anymore, it's confirmed working.
- Now we're moving off that CNN backbone onto a transformer. Be upfront about the tradeoff if asked: a transformer costs more compute and latency than VGG16, and we're deliberately taking that hit because it buys us meaningfully better accuracy on this exact task (this is the point most likely to get a question, see below).
- After the backbone swap, we add multimodal fusion (next slide covers exactly how).

### Slide 6: Proposed Model Architecture (~1:15)
- Walk the four-box flow left to right: high-precision image-to-coordinate data, into a transformer vision backbone (replacing VGG16), fused with IMU/magnetometer/gravimeter vectors, out to coordinates.
- Be explicit about staging: first get the transformer-only version working and matching (or beating) the VGG16 baseline, *then* add the multimodal fusion on top, not simultaneously.
- The dashed box is a placeholder. Say plainly that the detailed internal architecture (attention layout, fusion mechanism, i.e. concat vs. cross-attention, and where the sensor vectors enter the network) is still being finalized and will be in the endterm deck.

### Slide 7: Timeline (~0:40)
- Two phases only, ending April 2027; that's the actual constraint, not three phases through July.
- Phase 1 (now to Nov 2026, this semester): transformer backbone plus multimodal fusion, entirely in simulation.
- Phase 2 (Jan to Apr 2027): take that model and optimize it for speed and power on FPGA, then get it onto embedded flight hardware for a real UAV test.
- Say explicitly that Phase 2 isn't starting from zero: it directly extends a lab project from last semester that already proved FPGAs are highly power-efficient for this class of computation, so the hardware-efficiency question is largely de-risked going in.

---

## Likely questions to be ready for (1:00 Q&A)

- **"Why transformer over CNN if it costs more compute/latency?"** Because accuracy is the current bottleneck, not speed; Phase 2 (FPGA) is explicitly where the speed/power tradeoff gets clawed back later. We're sequencing accuracy first, efficiency second, on purpose.
- **"What does 'validated against the paper' mean exactly?"** We reran the reference implementation ourselves in simulation and our measured error matched the numbers reported in the CMU paper (no need to memorize exact figures, just confirm it was a real replication, not just reading the paper).
- **"What's the actual transformer architecture?"** Not finalized yet; that's the placeholder slide. If pressed, mention TransGeo/FSRA-style cross-view geo-localization transformers as the design family being drawn from (see refs [14], [15]).
- **"How will multimodal fusion actually combine the vectors with image features?"** Still an open design decision (concatenation vs. cross-attention); don't overcommit to one answer.
- **"Why is Phase 2 FPGA and not just a faster GPU?"** Power efficiency for onboard flight hardware; last semester's lab project specifically showed FPGA wins on power for this class of computation.
