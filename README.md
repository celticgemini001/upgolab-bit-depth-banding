# 8-bit vs 10-bit: why banding widens when you upscale

Measured notes on bit depth, gradient banding, dithering and H.264 bitrate.
Published at https://celticgemini001.github.io/upgolab-bit-depth-banding/

## What is measured

1. **Plateau width** — mean run of identical code values along a raw decoded row.
   An 8-bit ramp upscaled 4x gives 16.00 px; the formula `output_width/255` predicts
   16.06 px. Measured at x1/x2/x4/x8: 4.00 / 8.00 / 16.00 / 32.00 px, all within 0.5%
   of the prediction. Upscaling multiplies band width by the scale factor and never
   changes the number of bands.
2. **16-bit master into 8-bit delivery** — 16.00 px, identical to the 8-bit path.
   (Negative result: high internal precision cannot change the delivered plateau.)
3. **Dither** — Bayer 8x8 takes plateau 16.00 -> 2.00 px but RMSE 0.297 -> 0.408
   (+37%). Rounding to nearest is the RMSE-optimal quantiser by definition, so any
   dither must lose a per-pixel metric while winning perceptually.
4. **Late dither** (after the 8-bit step) — plateau 1.24 px, RMSE 0.766, 2.6x the
   control. Hides the edge, adds error, recovers nothing.
5. **Grain as dither** — 0.25% grain reaches plateau 1.67 px at 1.7x the RMSE of
   rounding; 1% reaches 1.13 px at 8.6x. A matrix is the cheaper noise source.
6. **Bit-depth ladder** — 8/9/10/12 bits give 256/509/969/1280 distinct levels at
   1280 px wide. 10-bit captures 76% of the available improvement.
7. **Real photograph** (1080x1620) — mean plateau 1.9 px, not the 4.2 px that
   `width/255` predicts, because texture breaks runs. 40.9% of pixels sit in runs
   longer than 4 px: those are the banding-prone regions.
8. **10-bit H.264 vs 8-bit** — 10-bit was *smaller* at CRF 12/18/23/28 (14.62x /
   7.72x / 1.53x / 1.01x). On noisy footage as a control the gap closed to -1.8% and
   -0.9%, i.e. a tie. The usual "20-50% more bitrate" did not reproduce in x264 CRF
   mode on either content type.

## Reproduce

`ffmpeg`, `numpy`. All counters read raw decoded frames, never screenshots. Full
snippet is at the bottom of the article.

## Related

Index of all measured guides: https://celticgemini001.github.io/upgolab-tools/

Browser upscaler: https://upgolab.com/en/upscale
