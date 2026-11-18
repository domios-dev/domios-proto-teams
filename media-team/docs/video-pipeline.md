# Video pipeline

Clips, covers and voice for posts, made from the real product.

## Steps

1. **Read the brief:** the message, channel and size, length, posting date, and the caption text or the latitude to write it. Find the real UI components that can show it. If the real product can't show it, say so before building, and never fake it. Things like a phone or a remote server need a real recording from the owner.
2. **Build the scene** from the product's real components with demo data, in a playground such as Vite or a storybook. Render stills at the key times and check them as images: overflow, menus outside the frame, wrong states.
3. **Capture frame-accurately:** a paused page clock, a headless browser and a fixed fps, typically 30.
4. **Finish:**
   - a music bed;
   - two-pass loudness normalisation to the target (−18 LUFS for feed clips);
   - H.264 with faststart;
   - burned-in captions that are large and readable without sound;
   - a hook in the first second.
5. **Verify** with ffprobe (length, size, codec), check frames at every caption beat, and measure the loudness.
6. **Register** the asset in the catalog with its specs and honest notes, including demo data and what wasn't checked. Report the asset ids and caveats to the Producer.

## Voice

- Generate a few short samples per language from existing account voices, including the owner's clone if one exists, and let the owner choose.
- Log every take (voice id, model, exact text) to a render log.
- Spell brand names phonetically for TTS. Pick takes nearest the reference voice in pitch and brightness.
- Keep a silent version next to any voiced cut.

## Rules

- **Show only real product UI.** No mock-ups, no private data, no internal names, tab ids or machine names.
- **Captions match what the product actually does.**
- **Never invent numbers or dates.** Mark demo values as demo in the notes.
- **Re-render at the same path and id when fixing.** Move media, never copy it, and never delete without the owner.
- **Verify with your eyes, not exit codes:** stills before the render, frames after it.
