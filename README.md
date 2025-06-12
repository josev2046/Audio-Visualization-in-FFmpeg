[![DOI](https://zenodo.org/badge/1000962745.svg)](https://doi.org/10.5281/zenodo.15650862)

This FFmpeg command transforms a standard MP4 file into a more engaging visual experience by dynamically overlaying an audio frequency spectrum. It achieves this by first generating a real-time visualization of the input audio's frequencies, then seamlessly compositing this transparent spectrum onto the original video stream, all while efficiently re-encoding the video and directly copying the original audio for optimal quality and playback. The final output is an MP4 optimised for immediate streaming, making audio-centric content visually captivating without compromising performance.


![image](https://github.com/user-attachments/assets/216b91b2-3a39-4d78-9ab6-a9ac503bf870)


### **Input File Specification**

The command begins by defining the source material.

* `ffmpeg -i input_audio_video.mp4`
    * `ffmpeg`: The primary command-line tool.
    * `-i`: Designates the subsequent argument as an **input file**.
    * `input_audio_video.mp4`: The path to your source MP4 file, containing both the original video and the audio stream intended for visualization.

---

### **Complex Filter Graph (`-filter_complex`)**

This is the core of the operation, where FFmpeg processes multiple streams simultaneously using a defined sequence of filters. The entire graph is enclosed in double quotes, and individual filter chains are separated by a semicolon (`;`).

* `"[0:a]showfreqs=s=1920x600:mode=line:fscale=log:colors=0xFFD700|0xFF4500|0x00FFFF|0x00FF00,format=yuva420p[spectrum]`
    * `[0:a]`: Selects the **audio stream** from the first input file (index `0`).
    * `showfreqs`: The filter responsible for generating the audio frequency spectrum visualization.
        * `s=1920x600`: Sets the **resolution** of the generated spectrum image to 1920 pixels wide by 600 pixels high. This determines the size of the overlay.
        * `mode=line`: Specifies the **display mode** for the spectrum as lines. The `bar` mode is an alternative.
        * `fscale=log`: Employs a **logarithmic scale** for frequency representation. This aligns with human perception of audio frequencies, enhancing the visibility of lower frequencies.
        * `colors=0xFFD700|0xFF4500|0x00FFFF|0x00FF00`: Defines the **hexadecimal RGB colours** used for the spectrum lines. Multiple colours are separated by a pipe `|`, enabling multi-coloured or gradient effects.
    * `,format=yuva420p`: Immediately applies the `format` filter.
        * `yuva420p`: Converts the spectrum output to this pixel format, crucially including an **alpha channel (`a`)**. The alpha channel is vital for the `overlay` filter to allow transparency, enabling the underlying video to show through areas not covered by the spectrum.
    * `[spectrum]`: Assigns a **label** named `spectrum` to the output of this filter chain, making it referenceable in subsequent parts of the graph.

* `; [0:v][spectrum]overlay=(W-w)/2:(H-h)/2[outv]"`
    * `;`: Separates the two distinct filter chains.
    * `[0:v]`: Selects the **video stream** from the first input file. This stream will serve as the background for the visual overlay.
    * `[spectrum]`: References the previously generated audio spectrum video stream. This stream will be placed as the foreground (overlay).
    * `overlay`: The video filter that superimposes one video stream onto another.
        * `(W-w)/2:(H-h)/2`: These expressions calculate the `x` and `y` coordinates for the overlay. `W` and `H` refer to the dimensions of the main (background) video, while `w` and `h` refer to the dimensions of the overlay video (`spectrum`). This calculation centres the `[spectrum]` overlay horizontally and vertically on the `[0:v]` stream.
    * `[outv]`: Assigns a **label** named `outv` to the resulting combined video stream, which will be mapped to the final output file.

---

### **Stream Mapping (`-map`)**

These options instruct FFmpeg which processed or original streams to include in the output file.

* `-map "[outv]"`
    * Maps the video stream labelled `[outv]` (the result of the `overlay` filter) to the primary video track of the output file.

* `-map 0:a`
    * Maps the **audio stream** from the first input file (`0:a`) directly to the primary audio track of the output file. This ensures the original audio is retained.

---

### **Video Encoding Options**

These parameters dictate how the processed video stream is compressed and formatted for the output.

* `-c:v libx264`
    * `-c:v`: Specifies the **video codec**.
    * `libx264`: Utilises the `libx264` encoder, a widely adopted and highly efficient H.264 video codec.

* `-preset medium`
    * Sets the `libx264` encoding **preset**. Presets govern the trade-off between encoding speed and compression efficiency/quality. `medium` provides a balanced compromise. Other options include `ultrafast`, `fast`, `slow`, `veryslow`, etc.

* `-crf 23`
    * Sets the **Constant Rate Factor (CRF)** for `libx264`. CRF is a quality-based encoding method where a lower value signifies higher quality (and larger file size), and a higher value indicates lower quality (and smaller file size). `23` is a commonly recommended value for good visual quality with reasonable file size.

* `-pix_fmt yuv420p`
    * Specifies the **pixel format** for the output video as `yuv420p`. This is a standard and highly compatible format for H.264, ensuring broad playback compatibility.

---

### **Audio Encoding Options**

These parameters define how the audio stream is handled for the output.

* `-c:a copy`
    * `-c:a`: Specifies the **audio codec**.
    * `copy`: This crucial optimisation instructs FFmpeg to **directly copy the audio stream** from the input to the output. This process is instantaneous and prevents any loss of audio quality or additional re-encoding time.

---

### **Output File Optimisation**

* `-movflags +faststart`
    * Optimises the output MP4 file for **web streaming** and progressive download. It repositions the `moov` atom (containing essential file metadata like duration and track information) from the end to the beginning of the file. This allows playback to commence almost immediately when the file is streamed or opened, rather than requiring the entire file to download first.

---

### **Output File Name**

* `Rebel_Archivists_11JUNE2025_dynamic_freq_spectrum_V3.mp4`
    * This is the designated **name for your final output video file**.
 


