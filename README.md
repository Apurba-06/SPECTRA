# SPECTRA - COMINT Signal Analysis Workbench (SIH 2026, PS 26147)

Blind analysis of `.wav` / `.iq` intercepts: every parameter is **measured from the samples**.

Pipeline: burst detection -> baud rate (cyclostationary) -> modulation classification (BPSK / QPSK / 16-QAM / 2-FSK) ->
matched filter + timing tracking -> carrier-offset + phase recovery -> soft demapping -> sync-word search
(all phase ambiguities / spectrum inversion) -> blind interleaver + FEC detection by trial decoding ->
soft Viterbi / Reed-Solomon / concatenated / LDPC decoding -> CRC-checked frame and payload.

## Run
    pip install -r requirements.txt
    python main.py                # GUI  (click Load File, then "Run Full Pipeline")
    python test_pipeline.py       # self-test: must print ALL PASS
    python demo_signals.py        # regenerate the demo captures

## Demo files (ground truth in *.truth.json)
| file | modulation | FEC | interleaver | IQ settings |
|---|---|---|---|---|
| demo.wav | QPSK 2400 baud | Conv K=7 | Block 16x16 | WAV |
| demo_bpsk_rs.wav | BPSK 1200 | RS(255,223) | none | WAV |
| demo_qpsk_concat.iq | QPSK 20 kbaud | RS + Conv | Forney depth 12 | 250000 Hz, cf32 |
| demo_fsk.wav | 2-FSK 1200 | Conv K=7 | Diagonal 8x32 | WAV |
| demo_16qam_uncoded.wav | 16-QAM 2400 | none | none | WAV |

## AI Analyst
Set your own key (never commit it):  `set GEMINI_API_KEY=your_key`  (or put `GEMINI_API_KEY=...` in a git-ignored `.env`).
Optional `GEMINI_MODEL` overrides the model name. Without a key the tab uses an offline rule-based analyst.

## Honest limits
Supported demodulators: BPSK, QPSK, 16-QAM, 2-FSK (8-PSK is detected only). FEC: conv K=7 r=1/2 (171,133),
RS(255,223) (reedsolo defaults, not the CCSDS dual-basis form), built-in LDPC n=1020. Pseudo-random interleavers
need a known seed. Sync library: CCSDS ASM and a few common markers - add yours in the hex box.

## BER vs Eb/N0 (Tab 8 and `python benchmark.py`)
Measured through the real receiver (RRC pulse, 120 Hz carrier offset, timing offset, AWGN -> timing/carrier recovery ->
soft demap -> decoder). Curves: uncoded QPSK (with theory overlay), Conv K=7 hard, Conv K=7 soft, LDPC soft.
Results are stored in `results/ber_vs_snr.png`, `ber_vs_snr_dark.png` (for slides) and `ber_vs_snr.csv`.
Frames with >30 % bit errors are counted as receiver lock failures (listed in the CSV, hollow markers), not as BER.
`python benchmark.py --quick` finishes in seconds; the Tab 8 buttons run the same code.
