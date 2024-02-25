# ComfyUI telegram bot

## Telegram bot for ComfyUI integration

Text2image, image2image work. There is LoRA support. Face replacement, stylization works via IPAdapter

Test bot: @stablecats_bot

## Installation and customization

A working installation of ComfyUI with additional modules is required:

- ComfyUI-Impact-Pack
- ComfyUI_UltimateSDUpscale

The installation of models for segmentation is also required: *face_yolov8m.pt*.

Upscaler: *4xNMKDSuperscale_4xNMKDSuperscale.pt*.

ControlNet Model: *control_v11f1e_sd15_tile.pth*

The default_lora.safetensors file from the assets directory should be copied to your LoRA directory. This is the workaround for the workflow without LoRA.


Rename the config.yaml.samlpe file to config.yaml and customize to your liking:
```
network:
  BOT_TOKEN: 'xxx:xxxxxxxx' - telegram bot token
  SERVER_ADDRESS: '127.0.0.1:8188' - ComfyUI API address.

bot:
  TRANSLATE: True - Whether to translate prompt languages to English (via deep_translate)
  DENY_MESSAGE: "Access denied" - Message if the user is not on the whitelist.
  HELP_TEXT: "Russian text can be used for generation.
By default, the caricature is created in 512x512 pixels resolution.
In the prompt you can specify the size WIDTHxHIGH. For example - 1024x512.
Commands:
/upscale .... - will create a high-resolution picture
/face .... - corrects facial defects"

comfyui:
  DEFAULT_MODEL: 'revAnimatedFp16_122.safetensors' is the default model name
  DEFAULT_VAE: 'vaeFtMse840000Ema_v10.safetensors' - default VAE model name
  DEFAULT_CONTROLNET: 'control_v11f1e_sd15_tile.pth' - ControlNet model for image2image
  SAMPLER: 'uni_pc' - the sampler used
  SAMPLER_STEPS: 30 - number of denois steps
  MAX_STEPS: 100
  TOKEN_MERGE_RATIO: '0.6'
  CLIP_SKIP: '-1'
  CONTROLNET_STRENGTH: '0.9'
  DEFAULT_WIDTH: 512
  DEFAULT_HEIGHT: 512
  MAX_WIDTH: 2048 - width limit     
  MAX_HEIGHT: 2048 - height limit
  BEAUTIFY_PROMPT: ',masterpiece, perfect, small details, highly detailed, best, high quality, professional photo' - added to prompt.
  NEGATIVE_PROMPT: 'low quality, worst quality, embedding:badhandv4, blurred, deformed, embedding:EasyNegative, embedding:badquality, watermark, text, font, signage, artist name, text, caption, jpeg artifacts' - customize negative prompt to your needs.
```

## Workflow Description

The workflows are used for generation, in the catalogs of workflows they are in ComfyUI API format

By default, when a clean prompt is received, a picture with dimensions DEFAULT_WIDTHxDEFAULT_HEIGHT is generated, the size can be specified in WIDTHxHEIGHT format.

The response from the bot will be two messages: a picture (with quality loss due to compression by telegram) and a PNG file with the original quality.

If the bot sends a picture, the picture will be converted according to the prompt, in this case `/face`, `/upscale` commands also work. Important! COntrolNet is used for img2img, not classical denois, which allows to give the result as close to the original as possible.

To add your own negative prompt instead of the built-in one - you can add `|` separator to the message.

Example of a full prompt (image2image) with sending a reference image:

`@rev #vlozhkin:0.4 $0.6 %50 768x512 /face pretty woman in red|blurred, bad quality`

- `@rev` - model selection (revAnimated)
- `#vlozhkin:0.4` - set LoRA with power 0.4
- `$0.6` - ControlNet coefficient (0 - none, 1 - maximum)
- `%50` - 50 sampler shaks
- `768x512` - input resolution
- `/face` - face enhancement and upsampling
- `pretty woman in red` - positive prompt.
- `|` - prompt separator
- `blurred, bad quality` - negative prompt

## Additional commands

`%50` - specify the number of sampler steps

`$0.5` - strength for image2image controlnet models

`/models` - list of models

`/loras` - list of available LoRAs



Original picture | Result
--- | ---
![Source Image](https://raw.githubusercontent.com/zlsl/comfyui_telegram_bot/main/examples/i2i_src.jpg) | ![Source Image](https://raw.githubusercontent.com/zlsl/comfyui_telegram_bot/main/examples/i2i_result.jpg) Cute demon with white snake scales, emerald eyes, sharp claws 1024x1024


The upload directory stores images sent to the bot

Catalog generated - generation results


## Commands

`/start`, `/help` - usage information from HELP_TEXT

`/upscale` - upscale ready picture

`/face` - face correction (each face on the picture will increase generation time)

`/me` - face photo setting (empty command for clearing), you can specify face weight by adding it to the command. For example /me 0.7

`/style` - setting the picture for styling (empty command for cleaning), you can specify the weight of cnbkz by adding it to the command. For example /style 0.7

## Restrict access

Whitelist is used, if whitelist in config.yaml is empty, then access is open to everyone. Add telegram uid of allowed users to the list.

```
whitelist:
  - uid1
  - uid2
```


## Using LoRA

In config.yaml, you need to add your lines to the `loras` section. LoRA is then connected by adding `#name_LoRA` to the prompt. For example `#vlozhkin`. You can specify the strength - `#lora_name:0.5`.

LoRA string format: `bot name`|`loRA model file name`|`strength by default`|`string to be added to the beginning of the prompt`.

Example:

```
loras:
  - ``vlozhkin|vlozhkin3.safetensors|1|vlozhkin style illustration''
```
