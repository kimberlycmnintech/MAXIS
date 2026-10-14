# Verification

## Current result

- Production build: passed (`npm run build`). The current 3D and chart bundles trigger Vite's advisory 500 kB chunk-size warning.
- Automated suite: **24 tests passed** across two files (15 simulator/workflow tests and nine UI interaction tests).
- Local preview: HTTP 200 at `http://127.0.0.1:5173/`.
- Browser visual verification: completed in the in-app browser at 1366×900, 375×812 and 812×375. Verified the live 3D overview, 2D schematic, equipment details drawer and responsive navigation; sampled water, energy, maintenance, alerts and resident pages. Resident mode contains no operator controls; checked mobile layouts have no horizontal overflow.

## Automated checks

The simulator tests in `src/data/store.test.ts` cover the single timer and subscriber cleanup, seeded/range-filtered history, 10,000 L capacity, flow/power integration, scenario correlation, invalid readings, command lifecycle and concurrency, stop latch, acknowledgment versus resolution, order receipt idempotency, maintenance supply use, local persistence/reset, equipment transitions, stream events and the complete normal → fault → mains → maintenance → recovery → rainwater story.

UI interaction checks in `src/App.test.tsx` verify navigation, workflows and keyboard interaction using a virtual DOM. The WebGL scene and responsive chart viewport are mocked in these tests; they do not assert GPU rendering or chart layout.

Run `npm test` and `npm run build` before handoff. The production build generates `dist/`; larger Three.js/chart bundles are expected for this prototype and the scene is loaded lazily.

## Browser checklist

- Inspect desktop layout at 1366 × 900, mobile at 375 × 812 and landscape at 812 × 375.
- Confirm 3D equipment labels, orbit/zoom/reset, scene highlighting and matching detail panels.
- Switch to 2D; select equipment and verify its readings.
- Confirm historical time ranges and visible gaps during Sensor Offline.
- Navigate without duplicated timers or fluctuating totals.
- Check resident restrictions and local issue reporting.
- Run the walkthrough in README.md; refresh after a workflow change and verify persistence.
- Verify readings stay legible and page width does not overflow on mobile.

The 3D view rendered in the in-app browser; check it once on the presentation laptop for its GPU/WebGL behavior. The interactive 2D fallback remains available when WebGL cannot render.
