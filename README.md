# Clear Display of LTR & RTL Languages on LED Matrix
This script guarantees accurate text display on LED matrixes, irrespective of the language, by utilizing the bidirectional algorithm or unidecode. It integrates flawlessly with AppDaemon and is compatible with the EspHoMaTriXv2 firmware, as well as any other LED Matrix firmware that supports custom fonts. **It’s also useful if you simply need to employ unidecode.**

**Key Features:**
- Supports all languages, including RTL scripts like Arabic, Hebrew, etc. Integrates `bidi` for proper bidirectional text handling.
- Leverages `unidecode` for special character conversion (e.g., "Björk" -> "Bjork").
- Seamless AppDaemon integration.

**How It Works:**
1. Monitors the media player's title in Home Assistant.
2. Identifies bidirectional text (RTL languages) or applies `unidecode` for special characters.
3. Converts and updates the text for LED matrix display.

**Examples:**
* **Bidirectional Algorithm:** "مرحبا بك" (Arabic for "Welcome") is correctly arranged for right-to-left display.
* **Unidecode:**
  *   Beyoncé: Converted to Beyonce due to the accented "é".
  *   Søren Kierkegaard: Converted to Soren Kierkegaard due to the Danish letter "ø".

**Getting Started:**
1. Install required libraries (`bidi`, `unidecode`).
2. Configure the AppDaemon script with your media player and sensor entities.
3. (Optional) For enhanced font support, consider EspHoMaTriXv2 firmware.

## Essential Components (in this example for Ulanzi Clock):
1. LED matrix (e.g., Ulanzi clock) https://a.aliexpress.com/_olGoYGb
2. EspHoMaTriXv2 firmware (https://github.com/lubeda/EspHoMaTriXv2)
3. Home Assistant (with add-ons)
4. ESPHome (add-on)
5. VSCode / File Editor (add-on)
6. AppDaemon (add-on)
7. Fonts
________________
## Installation 
1. Download the fonts and place them inside your ESPHome directory.
2. Get everything ready with EspHoMaTriXv2 - https://github.com/lubeda/EspHoMaTriXv2
3. Adjust the Led matrix YAML file with the new font and glyphs. 
For example adding Hebrew support:
```yaml
font:
  - file: hebpixel.ttf
    size: 16
    id: default_font
    glyphs:  |
      !?'"%&[]()+*=,-_.:°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyzאבגדהוזחטיכךלמםנןסעפףצץקרשת@$<>|\/
  - file: hebehmtx.ttf
    size: 16
    id: special_font
    glyphs:  |
      !?'"%&[]()+*=,-_.:°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyzאבגדהוזחטיכךלמםנןסעפףצץקרשת@$<>|\/
```
```yaml
ehmtxv2:
  default_font_id: default_font
  default_font_yoffset: 6
  special_font_id: special_font
```
4. Flash the **EspHoMaTriXv2** firmware
5. From the add-on store, install **AppDaemon.**
6. On the AppDaemon configuration page, add the **python-bidi** and **unidecode** package.
```yaml
system_packages: []
python_packages:
  - python-bidi
  - unidecode
init_commands: []
```
7. In the AppDaemon app directory (addons_config/appdaemon/apps), create a file named **bidiconverter.py** (with VSCode add-on) and paste the code (***Before pasting the code, make sure to adjust it to your personal needs***).

This script will execute each time the "media_title" attribute changes. Please update the "media_player" entity to match your specific entity name. In the provided example, the entity name is set as `media_player.era300`. Adjust this to reflect the actual entity name you are using.
```py
#bidiconverter.py
import re
import bidi
import time
from appdaemon.plugins.hass import hassapi as hass
from bidi.algorithm import get_display
from unidecode import unidecode

PLAYER = "media_player.era300"
SENSOR = "sensor.bidi"

class BidiConverter(hass.Hass):

    def initialize(self):
        self.listen_state(self.update_attributes, PLAYER, attribute='media_title')

    def update_attributes(self, entity, attribute, old, new, kwargs):
        for _ in range(100):  # Retry up to 100 times
            try:
                title = self.get_state(PLAYER, attribute="media_title") or ""
                album = self.get_state(PLAYER, attribute="media_album_name") or self.get_state(PLAYER, attribute="media_channel") or ""
                artist = self.get_state(PLAYER, attribute="media_artist") or self.get_state(PLAYER, attribute="media_title") or ""
                break  # Exit the loop if successful
            except:
                self.log("Failed to get media info, retrying...")
                time.sleep(0.5)
        else:
            self.log("Failed to get media info after 100 retries.")
            return

        # Convert to bidi or unidecode
        bidi_title = self.convert_text(title)
        bidi_album = self.convert_text(album)
        bidi_artist = self.convert_text(artist)
        
        new_attributes = {
            "media_artist": bidi_artist,
            "media_title": bidi_title,
            "media_album_name": bidi_album,
        }

        self.set_state(SENSOR, state="on", attributes=new_attributes)

    def convert_text(self, text):
        return get_display(text) if text and self.has_bidi(text) else unidecode(text) if text else ""

    def has_bidi(self, text):
            # covering Arabic, Hebrew and other RTL ranges
            bidi_regex = r"[\u0590-\u08FF]|[\u0600-\u06FF]|\u0700-\u074F|\u0750-\u077F|\u0780-\u07A6|\u08A0-\u08FF|\uFB50-\uFDFF|\uFE70-\uFEFF|\U00010E60-\U00010E7F|\U0001EE00-\U0001EEFF|\U0001F110-\U0001F5FF|\U00010F00-\U00010FFF|\u0621-\u06FF|\u0800-\u08FF|\u200E|\u200F"
            return bool(re.search(bidi_regex, text))
```
8. Open app.yaml file from the AppDaemon directory and add this code:
```yaml
bidiconverter:
  module: bidiconverter
  class: BidiConverter
```
9. Restart AppDaemon
## sensor.bidi
The next time a song is played, a new Home Assistant sensor named **sensor.bidi** will be generated. The sensor state will consistently be "on", and it should be disregarded.

The `sensor.bidi` sensor will include three attributes. It focuses primarily on right-to-left (RTL) languages while ignoring Latin characters and numbers. However, unidecode is applied when necessary to handle special characters within RTL languages. This ensures proper display even with non-standard characters. **This means that any given text string will be displayed correctly on the LED matrix strings, regardless of the language used.**
* **media_title:** Reflecting the title of the song.
* **media_artist:** Indicating the name of the artist.
* **media_album_name:** Describing the album name.

### Example Data sensor.bidi
```yaml
media_artist: Beyonce
media_title: םיתב העבראב ריש
media_album_name: (The Best of Beyonce) בטימה
```
When the orginal string was:
```
media_artist: Beyoncé
media_title: שיר בארבעה בתים
media_album_name: המיטב (The Best of Beyoncé)
```
### Example Automation
```yaml
description: "Display Song & Artist Name"
mode: single
trigger:
  - platform: state
    entity_id:
      - sensor.bidi
    attribute: media_title
condition: []
action:
  - service: esphome.ulanzi_rainbow_text_screen
    enabled: true
    data:
      default_font: true
      text: >-
        {{ state_attr('sensor.bidi', 'media_artist') }} - {{
        state_attr('sensor.bidi', 'media_title') }}
      lifetime: >-
        {{ (state_attr('media_player.era300', 'media_duration') | float(default=0) / 60) | int(default=1) if state_attr('media_player.era300', 'media_duration') is not none else 60 }}
      screen_time: 50
```
