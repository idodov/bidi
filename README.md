# RTL Language Display on LED Matrix (EspHoMaTriXv2) with Bidi Algorithm and AppDaemon
This script is a practical solution for displaying text on LED matrix screens. It uses the bidirectional algorithm and unidecode and integrates with AppDaemon (and EspHoMaTriXv2 in example beyond). Regardless of the language, the script ensures that any text string is correctly displayed on the LED matrix screens. It also converts special characters to their Latin equivalents. For example, names like Beyoncé and Björk are converted to Beyonce and Bjork, respectively. If you're only interested in the unidecode function for its ability to handle character encoding, you can disregard the instructions for installing fonts or making changes to the glyphs code.

The script monitors changes in a media player’s title in Home Assistant. If the title, album name, or artist name contains bidirectional characters (such as Arabic or Hebrew), the script converts the text for appropriate display. If not, it converts the text to ASCII. The converted text is then updated as attributes of a sensor entity in Home Assistant. This ensures that the text is correctly displayed in the Home Assistant interface when using LED Matrix screens.
### The Issue
When utilizing the LED display (such as the Ulanzi clock with ESPHome32), sending Latin-based text poses no problems. However, complications arise when the text string includes non-Latin characters like Hebrew, Arabic, Persian (Farsi), Urdu, Kurdish and Dhivehi (Maldivian). In such cases, the text displays in reverse order. The root cause is the lack of native support for these languages in the ESPHome32 system.
While reversing the text may resolve the problem, it introduces a new issue when the text includes numbers or Latin characters.
### My Use Case
For my personal use, when I am listening to music on a Sonos speaker via a music service, I display the artist's name and the song title on the clock. Some of the songs have Hebrew titles or artist names, and certain titles are a mix of Hebrew, Latin, or numerical characters, making simple string reversal impractical.
This script and method serve as my solution to address this issue. Please adjust the script according to your individual requirements.

The firmware I utilize is EspHoMaTriXv2, a highly recommended choice. This firmware seamlessly integrates with Home Assistant, and I selected it primarily because of its flexibility in allowing font customization. In the context of this project, I have incorporated two special Hebrew fonts. If you're seeking inspiration or additional fonts, you can explore more options on the following GitHub repository: https://github.com/trip5/Matrix-Fonts
## Things You Need
1. Ulanzi clock - https://a.aliexpress.com/_olGoYGb
2. EspHoMaTriXv2 firmware - https://github.com/lubeda/EspHoMaTriXv2
3. Home Assistant (with add-ons)
4. ESPHome (add-on)
5. VSCode (add-on)
6. AppDaemon (add-on)
7. Fonts
## Installation 
1. Download the fonts and place them inside your ESPHome directory.
2. Get everything ready with EspHoMaTriXv2 - https://github.com/lubeda/EspHoMaTriXv2
3. Adjust the Led matrix YAML file with the new font and glyphs. 
For example:
```yaml
font:
  - file: hebpixel.ttf
    size: 16
    id: default_font
    glyphs:  |
      !?'"%&[]()+*=,-_.:°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnÖÄÜöäüèéēøopqrstuvwxyzאבגדהוזחטיכךלמםנןסעפףצץקרשת@$<>|\/
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
            "media_artist_bidi": bidi_artist,
            "media_title_bidi": bidi_title,
            "media_album_name_bidi": bidi_album
        }

        self.set_state(SENSOR, state="on", attributes=new_attributes)

    def convert_text(self, text):
        return get_display(text) if text and self.has_bidi(text) else unidecode(text) if text else ""

    def has_bidi(self, text):
        return bool(re.search('[\u0590-\u08FF]', text)) if text else False
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

The "sensor.bidi" will include three attributes, focusing solely on right-to-left languages and ignoring Latin characters or numbers. 
**This means that any given text string will be displayed correctly on the LED matrix strings, regardless of the language used.**
* **media_title_bidi:** Reflecting the title of the song.
* **media_artist_bidi:** Indicating the name of the artist.
* **media_album_name_bidi:** Describing the album name.

### Example Data sensor.bidi
```yaml
media_artist_bidi: הטיפש
media_title_bidi: הסייד הלשיב אתבס
media_album_name_bidi: הסייד הלשיב אתבס REMIX
```
### Example Automation
```yaml
description: "Display Song & Artist Name"
mode: single
trigger:
  - platform: state
    entity_id:
      - sensor.bidi
    attribute: media_title_bidi
condition: []
action:
  - service: esphome.ulanzi_rainbow_text_screen
    enabled: true
    data:
      default_font: true
      text: >-
        {{ state_attr('sensor.bidi', 'media_artist_bidi') }} - {{
        state_attr('sensor.bidi', 'media_title_bidi') }}
      lifetime: >-
        {{ (state_attr('media_player.era300', 'media_duration') | float(default=0) / 60) | int(default=1) if state_attr('media_player.era300', 'media_duration') is not none else 60 }}
      screen_time: 50
```
