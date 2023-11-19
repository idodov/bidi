# RTL Language Display on LED Matrix with Bidi Algorithm and AppDaemon
A sample solution for rendering right-to-left languages on LED matrix screens, employing the bidirectional algorithm and leveraging AppDaemon

### The Issue
When utilizing the LED display (such as the Ulanzi clock with ESPHome32), sending Latin-based text poses no problems. However, complications arise when the text string includes non-Latin characters like Hebrew, Arabic (and other languages requiring the bidi algorithm). In such cases, the text displays in reverse order. The root cause is the lack of native support for these languages in the ESPHome32 system.
While reversing the text may resolve the problem, it introduces a new issue when the text includes numbers or Latin characters.
### My Use Case
For my personal use, when I am listening to music on a Sonos speaker via a music service, I display the artist's name and the song title on the clock. Some of the songs have Hebrew titles or artist names, and certain titles are a mix of Hebrew, Latin, or numerical characters, making simple string reversal impractical.
This script and method serve as my solution to address this issue. Please adjust the script according to your individual requirements.

The firmware I utilize is EspHoMaTriXv2, a highly recommended choice. This firmware seamlessly integrates with Home Assistant, and I selected it primarily because of its flexibility in allowing font customization. In the context of this project, I've updated two specific fonts to include Hebrew characters.
## Things You Need
1. Ulanzi clock - https://a.aliexpress.com/_olGoYGb
2. EspHoMaTriXv2 firmware - https://github.com/lubeda/EspHoMaTriXv2
3. Home Assistant (with add-ons)
4. VSCode (add-on)
5. AppDaemon (add-on)
