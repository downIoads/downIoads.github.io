---
title: "Using Stable Diffusion as Discord Emote Generator"
date: 2023-10-20T22:50:00+02:00
description: Short introduction to Stable Diffusion and example pictures.
draft: false
tags: [ai, text-to-image, windows]
---

## Introduction

Stable Diffusion is an open-source deep learning text-to-image model that does not have any filters and works offline. There are a few controversies around it (e.g., it has been trained on copyrighted artwork but gives the user of Stable Diffusion the rights to all images it spits out, and it does not have filters and can be used to create all sorts of weird content). It works offline and poses a threat to existing businesses that sell AI-generated images (e.g., OpenAI's closed-source DALLE-3). It is harder to use than, e.g., DALL-E because there are a lot of settings to adjust, which makes it highly customizable but also harder to use. I recently tried using Stable Diffusion again because [NVIDIA published new Game Ready drivers for their RTX GPUs, which greatly increases Stable Diffusion's performance](https://developer.nvidia.com/blog/unlock-faster-image-generation-in-stable-diffusion-web-ui-with-nvidia-tensorrt/).

The most accessible version of Stable Diffusion is the [Github repo of AUTOMATIC1111](https://github.com/AUTOMATIC1111/stable-diffusion-webui) which already has more than 100k stars (which makes it the most popular Github repository I have ever seen). Installing it on Windows is very simple and does not require any further knowledge.

## Generating Discord Emote

The unofficial Golang Programming Discord server lacks an ["Ackchyually"](https://knowyourmeme.com/memes/ackchyually-actually-guy) Gopher emote, so I thought trying to use Stable Diffusion to create such an emote could work. A Gopher is the mascot of the Golang programming language, and [this is what a Gopher looks like](https://www.svgrepo.com/show/373635/go-gopher.svg). The requirements are the following:
* It looks like a blue gopher (a gopher is also a rat animal)
* It wears glasses (the Ackchyually meme is based on someone being extra nerdy and knowledgeable, glasses seem like a good cliché)
* It should be a bit annoying to look at (you are supposed to use the emote when someone is technically right but annoying)

Below, you can see a screenshot of the Stable Diffusion WebUI:

![targets](/images/gopher-stable-diffusion/stable-diffusion-ui.png "Stable Diffusion WebUI")

There are lots of different sampling methods to choose from, so far, my favorite has been "Euler a". It depends on the usecase, but for generating Gopher-like emotes, this one worked the best for me and was fairly consistent (it heavily depends on the prompt). Creating a prompt that works is just trial-and-error; there are people who pretend that writing good prompts should be a paid job. I disagree with this, as currently these text-to-image models are blackboxes that no one fully understands. It's ridiculous to pretend to be an expert at generating good images. While it is true that the prompt itself (and other settings) affect the resulting images, there is no proven "correct" way to write prompts. Anyone gets better by trial-and-error, and I don't think anyone should call themselves "professional prompt engineers". It seems like a pseudoscience to me.

## Examples prompts and images

* Prompt: blue rat, two long front teeth, cute painting, nerd wearing human glasses, face closeup
* Settings: Steps: 30, Sampler: Euler a, CFG scale: 10, Seed: 1643617518, Size: 512x512, Model hash: 6ce0161689, Model: v1-5-pruned-emaonly, Version: v1.6.0

![targets](/images/gopher-stable-diffusion/akshually1.png "First gopher")

---

* Prompt: blue rat, two long front teeth, cute painting, nerd wearing human glasses, face closeup
* Settings: Steps: 30, Sampler: Euler a, CFG scale: 10, Seed: 1604019645, Size: 512x512, Model hash: 6ce0161689, Model: v1-5-pruned-emaonly, Version: v1.6.0

![targets](/images/gopher-stable-diffusion/akshually2.png "Second gopher")

---

* Prompt: blue rat, painting drawn by claude monet, nerd wearing human glasses, face closeup, scruffy and unkept
* Settings: Steps: 36, Sampler: Euler a, CFG scale: 11, Seed: 1228714833, Size: 512x512, Model hash: 6ce0161689, Model: v1-5-pruned-emaonly, Version: v1.6.0

![targets](/images/gopher-stable-diffusion/corpa-gopher.png "Corpa gopher")

---

* Prompt: two long front teeth blue rat, visible, blue sharpie drawing, nerd wearing human glasses, face closeup
* Settings: Steps: 30, Sampler: Euler a, CFG scale: 12, Seed: 3241088859, Size: 512x512, Model hash: 6ce0161689, Model: v1-5-pruned-emaonly, Version: v1.6.0

[![targets](/images/gopher-stable-diffusion/akshually3.png "Third gopher")](/images/gopher-stable-diffusion/akshually3-upscaled.png)

---

I really like the last image. It captures the essence of the "Ackchyually" meme because the Gopher looks both knowledgeable and annoying. With this image, I decided to test Stable Diffusion's upscale capability. Since it is recommended to generate images at 512x512, you can then upscale these images to high resolutions like 3784x2976. I manually edited the upscaled version of the image above to remove the background before upscaling it SwinIR_4x (which can be found under Extras-ScaleTo-Upscaler1). If you click on the image above, you can see the high-resolution result. Now, anyone can make a nice Discord emote from this image. I tried further prompts with "Renee French style" and "Claude Monet style", which also resulted in decent images. 

## Summary

Modern AI tools are very exciting, and they will only get better from now on. Uncle Ben once said: "With great power comes great responsibility" and Stable Diffusion is a good example of this. While it can be used to create disturbing content, the CEO of Stability AI is convinced that putting the capabilities of Stable Diffusion into the hands of the public will result in the technology providing a net benefit, in spite of the potential negative consequences. I agree with this, and I hope that other AI technologies from other areas, like e.g., open-source LLMs, can catch up with closed-source models like OpenAI's GPT-4 so that everyone can have free access to these amazing technologies.