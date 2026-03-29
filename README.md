# ESPHome + Home Assistant E-Paper Dashboard

A simple, low-power dashboard using ESPHome and Home Assistant, displayed on a 7.5" e-paper device. It shows weather, temperatures, service status, and Proxmox container stats, cycling through splash, home, and Proxmox screens.

![IMG_8023](https://github.com/user-attachments/assets/8d11993c-9ca8-453f-935b-2174f4e9778e)![IMG_8025](https://github.com/user-attachments/assets/ed1c2b3c-29d5-497b-98e4-d833ddbbaff3)


---

## Purpose

Provide key home and system data at a glance—without needing to unlock or tap anything. It’s always visible, silent, and easy to customize.

---

## Key Features

- 3 auto-rotating screens:  
  1. Splash image  
  2. Weather, indoors/outdoors, service health  
  3. Proxmox container resource usage (CPU, memory, swap)  
- Clean layout with Material Design icons  
- Very low energy use—no backlight, always readable  
- Runs on a Seeed Studio **XIAO 7.5" ePaper Panel**, fully integrated and easy to install

---

## Hardware Overview

The project uses the [Seeed Studio XIAO 7.5" ePaper Panel](https://www.seeedstudio.com/XIAO-7-5-ePaper-Panel-p-6416.html?srsltid=AfmBOoo7yNJdh6ocDCPpBVsW7EUONfYskGAK5dhxONjg4-Wjx3BBmSTa), which includes:

- Built-in **XIAO ESP32-C3**, Wi-Fi capable  
- 7.5" 800×480 e-paper display  
- USB-C port, boot/reset buttons, and optional battery  
- Tight integration with ESPHome and Home Assistant via their wiki guide :contentReference[oaicite:0]{index=0}

Note: I plan to paint the panel matte black to blend with my setup.

---

## Setup Instructions

### 1. Install ESPHome in Home Assistant

- Go to **Settings → Add-ons → Add-on Store**
- Find ESPHome, click **Install**, then **Start**
- If it’s missing, you may need a supported Home Assistant installation. For Home Assistant Container, use ESPHome Device Builder via Docker :contentReference[oaicite:1]{index=1}.

---

### 2. Add a New Device in ESPHome

- Select **New Device**, choose a name (e.g., `epaper_dashboard`)
- Pick ESP32-C3 as the board, then click **Edit**

---

### 3. Flash the Dashboard

- Replace the auto-generated YAML with `dashboard.yaml` from this repo
- Update Wi-Fi and Home Assistant API credentials
- Customize any sensor IDs to match your Home Assistant setup

#### Flashing Options:
- **USB (initial setup)**: Click **Install → Manual Download**, then use the web uploader to flash the panel
- **OTA (subsequent updates)**: After initial setup, updates can be pushed wirelessly if `ota:` and `api:` are properly configured

For full ESPHome setup and examples, see the official [Seeed wiki](https://wiki.seeedstudio.com/xiao_075inch_epaper_panel_esphome/) .

---

## Customization

This dashboard reflects my setup—but you can adapt it:

- Add different sensors (e.g., energy usage, security status)  
- Re-order or combine slides  
- Change icons, fonts, or layout elements - I got all the icons using the MDI font, and used this [site](https://pictogrammers.com/library/mdi/) to browse through icons.

---

## About the Code

The ESPHome code driving this display is fairly manual and hardcoded. Every element—icons, positions, font sizes, sensor mapping—is placed by hand, which makes it tricky to update or expand. There’s no dynamic layout or auto-scaling, so small tweaks often require testing and adjusting coordinates little by little.

It works for me as it is now, and I don't expect to need frequent changes. But it's definitely not plug-and-play, and there's plenty room for improvement.

I’d like to eventually improve the layout logic or even explore a different approach entirely—perhaps switching to Arduino and making direct API calls to Home Assistant for more flexibility. For now, this solution fits my needs, and it stays reliable once configured. Just be prepared for some trial and error if you decide to build your own version.

---

## Best Practices (ESPHome + ePaper + Puppet)

- **Use the correct panel model first**: if polarity looks inverted (black background / white text), verify the exact `waveshare_epaper` model for your hardware before changing URL colors.
- **Gate downloads on Wi-Fi connectivity**: trigger initial `online_image` updates from `wifi.on_connect` (with a short delay) so startup races do not cause HTTP failures.
- **Keep RAM headroom on ESP32-C3**: avoid unnecessary components (extra fonts/web UI), keep logger level conservative, and tune `online_image.buffer_size` based on real heap behavior.
- **Use binary BMP for e-paper**: keep `format=bmp` and `bmp_mode=binary` for predictable black/white rendering and smaller decode complexity.
- **Retry only while awake**: for battery-powered setups, use short retry intervals during the awake window and deep sleep after success or timeout.
- **Update display after successful download**: drive refresh from `online_image.on_download_finished` so the panel does not redraw stale/blank frames.
- **Prefer full refresh for ghosting control**: for supported models, set `full_update_every` to reduce artifacts over time.
- **Validate every YAML revision**: run `esphome config` (or at least YAML parsing) before flashing, because unsupported options vary by model.

---

## Repo Contents

- `dashboard.yaml` – ESPHome configuration  
- `esphome/puppet_xiao75.yaml` – ESPHome config for rendering a Home Assistant Puppet Lovelace view on the XIAO 7.5" panel  
- `fonts/` – Font files including Material Design icons  
- `image/` – Boot splash image (optional), logos, etc 

**REMINDER: For any images to be displayed nicely on a binary e-ink display, remember to reformat it as a binary image using a tool such as this one: https://onlinejpgtools.com/create-binary-jpg



---

## License

Feel free to use, modify, and build on this—no restrictions.

---


## XIAO 7.5" ePaper Panel specs

| Item | Description |
|---|---|
| MCU | XIAO ESP32-C3 |
| Display | 7.5" Monochrome ePaper Display |
| Resolution | 800 x 480 |
| Battery | 2000mAh |
| Dimension | 178 x 131 x 19 mm |
| Weight | 218g |
| Operating Temperature | -25°C to 50°C |
| Power Supply | USB Type-C 5V |
| Enclosure | 3D Printing (PLA) |

## New starter templates (photo-style card layout)

This repo now includes ready-made template files for the Seeed Studio XIAO 7.5" ePaper panel with a 4x2 card layout similar to the reference photo:

- `templates/esphome_xiao_75_epaper_base.yaml` (minimal ESPHome base config with panel defaults)
- `templates/esphome_xiao_75_dashboard_template.yaml` (photo-style card dashboard ESPHome config)
- `templates/home_assistant_epaper_package.yaml` (Home Assistant template sensors package)

These templates are pre-mapped to the real entities shared for E-Bike charger, electricity rate/session cost, and wall fairy light metrics.
