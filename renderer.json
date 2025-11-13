import express from "express";
import { exec } from "child_process";
import cors from "cors";
import fs from "fs";
import fetch from "node-fetch";

const app = express();
app.use(cors());
app.use(express.json({ limit: "50mb" }));

app.post("/render", async (req, res) => {
  const { 
    audio_url,
    hook_line_1,
    script_1,
    script_2,
    script_3,
    quote,
    closer
  } = req.body;

  const audioPath = `/tmp/audio_${Date.now()}.mp3`;
  const outputPath = `/tmp/output_${Date.now()}.mp4`;
  const bgPath = "/opt/render/project/src/background.mp4";

  // Download audio
  const response = await fetch(audio_url);
  const buffer = await response.arrayBuffer();
  fs.writeFileSync(audioPath, Buffer.from(buffer));

  // FFmpeg command
  const command = `
    ffmpeg -i ${bgPath} -i ${audioPath} -filter_complex "
    drawtext=text='${hook_line_1}':fontcolor=white:fontsize=60:x=(w-text_w)/2:y=200:enable='between(t,0,3)',
    drawtext=text='${script_1}':fontcolor=white:fontsize=48:x=(w-text_w)/2:y=400:enable='between(t,3,7)',
    drawtext=text='${script_2}':fontcolor=white:fontsize=48:x=(w-text_w)/2:y=400:enable='between(t,7,11)',
    drawtext=text='${script_3}':fontcolor=white:fontsize=48:x=(w-text_w)/2:y=400:enable='between(t,11,15)',
    drawtext=text='${quote}':fontcolor=white:fontsize=40:x=(w-text_w)/2:y=650:enable='between(t,15,19)',
    drawtext=text='${closer}':fontcolor=white:fontsize=52:x=(w-text_w)/2:y=1650:enable='between(t,19,23)'
    " -map 0:v -map 1:a -c:v libx264 -c:a aac -shortest ${outputPath}
  `;

  exec(command, (error) => {
    if (error) {
      return res.status(500).send("FFmpeg render error");
    }

    const video = fs.readFileSync(outputPath);
    res.setHeader("Content-Type", "video/mp4");
    res.send(video);

    fs.unlinkSync(audioPath);
    fs.unlinkSync(outputPath);
  });
});

app.listen(10000, () => console.log("Renderer running on port 10000"));
