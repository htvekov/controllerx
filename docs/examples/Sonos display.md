


### Sonos/Symfonisk display unit with ESPHome v1.1

![Three different Sonos displays](https://github.com/htvekov/controllerx/blob/master/docs/assets/img/Sonos_displays_1.jpg)

Using ControllerX - Controlling your Sonos speakers have never been easier 😎 But the occasional wish for a visual check on what’s actually playing, volume setting, media artist/title, active speakers in group etc. is still there 👀 Leaving you with no other choice than adding a display integration as the obvious solution for this need 😉

All it takes is an ESP8266 with ESPHome software, an appropriate display, a handfull (well actually 2 handfulls and then some) of HA sensors and you’re all set to go 🚀 ESPHome is a system framework for ESP8266 units that has support for several I2C OLED/E-ink display types and numerous sensors, has direct HA integration via add-on and easy, yet powerful YAML configuration. Read more about ESPHome and how to set it up in HA here: https://esphome.io/

### Hardware:
I initially used the simple and inexpensive (less than 2 US$ ) SSD1306 0,96" OLED display for this build. Resolution is only 128x64. But still enough, when using several pages to be displayed continously. The SSD1306 has a 'big brother' in the SSD1309 display. This display has identical resolution as SSD1306, is priced at some 14 US$, can use same drivers/library as SSD1306 but is much, much larger at 2,42". I really like this good sized and simple I2C display and ended up using this display in the final build, as it's much easier to read from a distance 🙂

An optional PIR sensor or microwave radar sensor can be added for automatic dimming (brightness control) and turning the display on/off completely. The RCWL 0516 sensor is cheap, but can be somewhat difficult to use in 'tight' builds as it's somewhat sensitive to many things - WiFi in particular. So you could experience some false triggers using this sensor if fitted very close to the ESP8266. AM 312 is a cheap and simple 'no nonsense' PIR sensor that just always works as expected. Sensor can also be used for other purposes as well in HA 🙂 Sensor is configured in YAML using pin D5 (GPIO14). Display is set to dim down after 5 minutes with no PIR triggers and completely off after additional 25 minutes without registered movement.

Please note that (at least on my display version) SSD1309 display can't be turned completely off with brightness = 0, but this works perfectly on my SSD1306 display. Instead SSD1309 has to be turned on/off with specific commands. Check what works on your specific display and revise implementation method/ESPHome yaml config code accordingly.

**Revised v1.1
Not done yet !!
Opdater font - 6 nye symboler (Wifi, alarm, album + 3 x højttaler)
Glyphs er opdateret i ESP ssd1309
Check orig. ttf font (download fra github og lav den igen) - der ER forskel
Der er noget galt med hovedet på person for artist
Behold 'gammel' font og reset glyphs. Evt. opdater med nye højttaler symboler. Der er alligevel ikke plads til flere sensorer uden ESP crasher. 

Opdater med turn_on() turn_off() commands **done in ssd1306**
Opdater med dynamisk brightness(display_name): 0-1 (float med decimaler) **done in ssd1306**

Husk at nævne eksempel med flash light ved Sonos reset to default.

backly i ESPHome sender ikke on/off display hvis den ikke selv har skiftet state !! Altså kørt en full cycle ?? OMskrevet for længst. **Check om det er ok nu ?**

**Mangler:**
Condition for display content with PIR & Sonos active playing (skal tage højde for at PIR er optional) Burde ikke være nødvendigt, men husk at sætte display on ved ESP opstart. virker pt. ssd1306 ok.


Add needed ESPHome version. Dev. at the moment!!

Virker nu via tweaks. Sæt i2c speed to min. 100khz (her 200khz)
On boot virker ikke til at sætte display on (crash) når der er to tællere backly og backly 1.
Så første gennemløb i display page, kigger efter page=0 og tænder så displayet den ene gang.


Display active on: Så virker PIR sensor og kan tænde og slukke displayet. Dynamisk med 2 forskellige timeouts. pt. 5 sec. dæmpning og 10 sec. sluk helt **ssd1306 ok**

Display active off: Display kan ikke tændes ved PIR action **ssd1306 ok**

Shuffle symbolet er midlertidigt i linie 1. Skal det være der ?
SKal kun bruges ved playlister så linie 1 er egentlig ok.


**Færdig implementeret:**
**HA sensors:**
display active PIR  (on/off) eller bare display on/off hvis ingen PIR
HA connected status
Wifi strength
reboot ESP

Display:
**display idle "stop" symbol** 
**display Mute volume symbol**
**script kørsel af identisk del af display lambda !! ssd1306 ok**



Some links examples for hardware below. These are just some random sellers I've picked. Not necessarily the cheapest or best sellers.

[Wemos D1 mini – ESP8266](https://www.aliexpress.com/item/32845253497.html)

[0,96" 12864 SSD1306 OLED display](https://www.aliexpress.com/item/32896971385.html)

[2,42” 12864 SSD1309 OLED display
(direct replacement for the much smaller 0,96” SSD1306 display and can use same library)](https://www.aliexpress.com/item/33024448944.html)

[Alternatively you can use a Wemos NodeMCU ESP8266 with integrated 0,96" OLED display](https://www.aliexpress.com/item/4000287451981.html)

**Optional sensors to dim display when no movement is detected.**
[Movement sensor 1 - RCWL 0516 Microwave Micro Wave Radar Sensor Board](https://www.aliexpress.com/item/33011567518.html)

[Movement sensor 2 - AM312 # PIR Motion Human Sensor](https://www.aliexpress.com/item/32921030810.html)

**One note on the SSD1309 display**<br />
In order to get it to work as I2C instead of SPI, you need to do a bit of soldering. On the specific display type I bought, you need to bridge (short) R5 and move R4 to R3. Remember that display will NOT work unless RES is connected to RST on ESP8266 (or any available pin and controlled in ESPHome sw). Note: Display only supports 3,3v on VCC. Some have reported that display tolerates 5v. I wouldn’t take that risk, though! I’ve kept both CS and DS ‘floating’. Haven’t had any I2C address issues so far. Use pull-up/down resistors if you experience issues.

**Connections:**
**SSD1306/1309 --> Wemos D1 mini**

    VCC: 	3,3v !!! (SSD1306 only: 3,3v-5v)
    GND: 	GND
    SDA: 	D1
    SCL: 	D2

**SSD1309 only --> Wemos D1 mini**

    RES:	D0 or RST
    CS :	NC (No Connection - 'floating'. Default I2C address 0x3c)
    DC :	NC (No Connection - 'floating')


### Display setup:
My current display setup consists of four pages that all are displayed for 5 seconds.

Following information is displayed on the screen:

**All pages:** Source (if not present, display ‘Sonos/Playlist’)
						volume setting and play/pause/idle status.

**Page 1:** Active master/slave speakers.

**Page 2:** Media artist/media title
					(if not available from stream, display time instead)

**Page 3:** Time

**Page 4:** Outdoor temperature sensor value

### True Type Fonts:
Three 'standard' Calibri TT fonts are used plus a 'special' version of Heydings Icons font in which I've included some Heydings Controls icons as well.
If you experience some strange characters on the display, you probably need to edit the glyphs in ESPHome YAML and add whatever language specific characters you find are missing. 

Calibri TTF fonts link: [https://www.fontdload.com/dl/calibri-font/](https://www.fontdload.com/dl/calibri-font/)

Heydings Icons special file link: [https://github.com/htvekov/controllerx/tree/master/docs/assets/img/HeydingsIconsSymbols.ttf](https://github.com/htvekov/controllerx/blob/master/docs/assets/img/HeydingsIconsSymbols.ttf)

Copy Calibri Bold, Calibri Regular, Calibri Light fonts plus the special Heydings Icons font file to the ESPHome folder `/config/esphome/`

### Home Assistant sensors:
Below you’ll find the HA template sensors needed in `configuration.yaml `for ESPHome display to work. ESPHome will establish some four HA sensors as well, presented on HA frontend: PIR sensor, connection status, WiFi strength and display on/off sensor. If display on/off is turned off from HA, then triggering PIR will not turn on display or alter brightness.

Note: `media_artist` and `media_title` attributes from HA's Sonos integration *could* be swapped for some radio stations, as these attributes are split from one combined string in the stream. Some radio stations have artist - title order, others use title - artist. You really can't tell...
My danish radio stations (source list) all use the 'swapped' version, so my templates below swap these two attributes for radio stations. 

Enter your master speaker as trigger entity ID for all templates but the first two (search for `media_player.office` and replace with your master speaker entity). Without this specific hardcoded trigger entity, templates simply doesn't always update correctly. So unfortunately they're needed for now, until I hopefully find a 'cleaner' and more dynamic solution.

```yaml
# Sonos sensors
sensor:
  - platform: template
    sensors:
      sonos_master_friendly:
        friendly_name: "Sonos Master Friendly"
        entity_id: group.sonos_all
        value_template: "{{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'friendly_name') }}"
      sonos_slaves_friendly:
        friendly_name: "Sonos Slaves Friendly"
        entity_id: group.sonos_all
        value_template: >- 
           {% for entity_id in state_attr("group.sonos_all", "entity_id")[1:] -%}
             {% set friendly_name = state_attr(entity_id, "friendly_name") %}
             {%- if loop.last %}{{ friendly_name }}{% else %}{{ friendly_name }}, {% endif -%}
           {%- endfor %}
      media_title: # Swap title/artist if 'source' attribute is not present = radio
        entity_id: media_player.office # Sonos master speaker
        value_template: >-
          {% if is_state('sensor.media_source' , "no source") %}
            {{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'media_title') }}
          {% else %}
            {{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'media_artist') }}
          {% endif %}
      media_artist: # Swap title/artist if 'source' attribute is not present = radio
        entity_id: media_player.office # Sonos master speaker
        value_template: >-
          {% if is_state('sensor.media_source' , "no source") %}
            {{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'media_artist') }}
          {% else %}
            {{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'media_title') }}
          {% endif %}
      media_album_name:
        entity_id: media_player.office # Sonos master speaker
        value_template: "{{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'media_album_name') }}"
      media_source: # Remove all after 'DR P4 Fyn' as source (to fit on display)
        entity_id: media_player.office # Sonos master speaker
        value_template: >-
          {% if state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'source') %}
            {{states.media_player.office.attributes.source.split('96.8')[0]}}
          {% else %}
            no source
          {% endif %}
      volume:
        entity_id: media_player.office # Sonos master speaker
        value_template: "{{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'volume_level')|float * 100 }}"
      sonos_master_group_entities:
        entity_id: media_player.office # Sonos master speaker
        value_template: "{{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'sonos_group') }}"
```

### Home Assistant group:
Here you define your Sonos speaker entities. Master speaker has to be entered as first entity and all that's actually needed. Active slave speakers will dynamically be added on HA restart or when group configuration is changed (via Sonos app/HA service calls eg.) If you're only using one speaker, you still need to create the group in `groups.yaml` and populate with that single master speaker entity, as the group entity is needed in the code.

**One note on master speaker, slaves and Sonos groups**<br />
Your defined master speaker actually doesn't need to be ***the*** master speaker. As long as it's part of the group (master ***or*** slave), then display will still show data for the group. But if defined master speaker is removed from the group, it will be a 'single speaker group' on it's own, and display will reflect master speaker data only. 
```yaml
sonos_all:
  name: sonos_all
  entities:
    - media_player.office     # This HAS to be your MASTER speaker
  #	- media_player.kitchen    # Optional - SLAVE speaker #1
  #	- media_player.livingroom # Optional - SLAVE speaker #2
```

### Home Assistant automations:
First automation is identical with the one I've already used in my ControllerX Sonos group setup example: ([https://xaviml.github.io/controllerx/examples/sonos](https://xaviml.github.io/controllerx/examples/sonos))

Second automation is purely optional, and not really directly related to the display. It's just a quick shortcut to easily reset active speakers within group, volume and source playing to some defaults you've defined in the automation. Really nice when you have teenagers in the house, messing with active speaker entities in the group, playlists and volume all the time... 😉
The automation is written for an Ikea E1810 remote with z2m ControllerX HA integration. Here `toggle_hold`(Press and hold center button for appx. 3.5 seconds) is used as trigger.

```yaml
- id: Dynamic Sonos groups
  alias: Dynamic Sonos groups
  trigger:
    platform: state
    entity_id: sensor.sonos_master_group_entities # Same as defined in configuration.yaml
    platform: homeassistant
    event: start
  action:
  - service: group.set
    data_template:
      object_id: sonos_all # Name of Sonos group in groups.yaml
      entities: "{{ state_attr(state_attr('group.sonos_all', 'entity_id')[0], 'sonos_group') | join(',') }}"

- id: Sonos reset to defaults settings
  alias: Sonos reset to defaults settings
  trigger:
    platform: state
    entity_id: sensor.your_E1810_sensor_action
    to: 'toggle_hold'
  action:
  - service: sonos.join
    data:
      master: media_player.office # This HAS to be your MASTER speaker
      entity_id:
      - media_player.kitchen # SLAVE speaker #1
      - media_player.livingroom # SLAVE speaker #2
  - service: media_player.volume_set # Reset volume to 25 for all speakers
    data_template:
      entity_id:
      - media_player.office
      - media_player.kitchen
      - media_player.livingroom
      volume_level: 0.25
  - service: media_player.select_source # Reset to your default choice of source
    data:
      entity_id: media_player.office # This HAS to be your MASTER speaker
      source: 'DR P4 Fyn 96.8 (Nyheder)'
  - service: media_player.media_play # Start playing
    entity_id: media_player.office # This HAS to be your MASTER speaker
```


### ESPHome YAML configuration:
As ESPHome currently don't support attributes, all data to be displayed has to be in separate HA sensors. Hence the huge amount of sensors.

If you're not using a movement sensor in your build, you could, but actually don't need to, revise YAML. If you experience issues with the 'floating' GPIO used for the PIR sensor, just pull pin D5 permanently low or high.

Two entities needs to be entered. Your Sonos master speaker and optional temperature sensor.
If temperature sensor is omitted, you can just delete page 3 & 4 from the display configuration.

Revise  `sonos_status` and `outdoor_temp` sensors in YAML below, to match your HA entities for Sonos master speaker and outdoor temperature sensor. Create a new ESPHome node and configure it with your WiFi credentials. Edit node and copy/paste revised YAML below (from `time:` and onwards) to your node. Save it, upload and enjoy! 🎉😎

**One final note on current YAML configuration**
ESPHome is at **max** with all these sensors, schedulers running and an extensive display lambda. Addding just one extra sensor to current YAML, will make ESPHome crash on boot. Omitting  `fast_connect: true`from WiFi configuration in YAML will also send ESPHome into an eternal stack trace error boot loop 🚀💀 
So 'tweak' YAML with care! 😁😉 
```yaml
esphome:
  name: ssd1309
  platform: ESP8266
  board: d1_mini

wifi:
  ssid: "your ssid"
  password: "your password"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "Ssd1309 Fallback Hotspot"
    password: "Autogenerated password"
captive_portal:

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:

time:
  - platform: homeassistant
    id: esptime

sensor:
    # Outdoor temperature sensor - only used in lambda page 4
  - platform: homeassistant
    id: outdoor_temp
    entity_id: sensor.your_temperature_sensor
    internal: true
  - platform: homeassistant
    id: sonos_volume
    entity_id: sensor.volume
    internal: true

text_sensor:
    # Sonos master speaker
  - platform: homeassistant  
    id: sonos_status
    entity_id: media_player.your_master_speaker
    internal: true
  - platform: homeassistant  
    id: media_source
    entity_id: sensor.media_source
    internal: true
  - platform: homeassistant
    id: media_artist
    entity_id: sensor.media_artist
    internal: true
  - platform: homeassistant
    id: media_title
    entity_id: sensor.media_title
    internal: true
  - platform: homeassistant
    id: sonos_master
    entity_id: sensor.sonos_master_friendly
    internal: true
  - platform: homeassistant
    id: sonos_slaves
    entity_id: sensor.sonos_slaves_friendly
    internal: true
    
font:
  - file: "Calibri Bold.ttf"
    id: font_large
    size: 23
    glyphs: '!%"()+,-_.:°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz█£$@æøåÆØÅ&#''´’?üöäé'
  - file: "Calibri Regular.ttf"
    id: font_medium
    size: 19
    glyphs: '!%"()+,-_.:°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz█£$@æøåÆØÅ&#''´’?üöäé'
  - file: "Calibri Light.ttf"
    id: font_small
    size: 11
    glyphs: '!%"()+,-_.:°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz█£$@æøåÆØÅ&#''´’?üöäé'
  - file: "Calibri Bold.ttf"
    id: font_small_bold
    size: 12
    glyphs: '!%"()+,-_.:°0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ abcdefghijklmnopqrstuvwxyz█£$@æøåÆØÅ&#''´’?üöäé'
  - file: "HeydingsIconsSymbols.ttf"
    id: font_icons_small
    size: 11
    glyphs: '067HADJabjsmx'
  - file: "HeydingsIconsSymbols.ttf"
    id: font_icons_medium
    size: 19
    glyphs: '067HADJabjsmx'
  - file: "HeydingsIconsSymbols.ttf"
    id: font_icons_14
    size: 14
    glyphs: '067HADJabjsmx'
  
i2c:
  scl: D1 # SCL - SSD1306/1309
  sda: D2 # SDA - SSD1306/1309
  
interval:
  - interval: 5s # Display period for each page
    then:
      - display.page.show_next: sonos
      - component.update: sonos

display:
  - platform: ssd1306_i2c
    model: "SSD1306 128x64"
    reset_pin: D0 # RST - SSD1306/1309. Used to turn on/off display
    address: 0x3C # Default I2C adress
    brightness: 100% # Select brightness
    id: sonos
    pages:
      - id: page1
        lambda: |-
          // it.rectangle(0, 0, 128, 64); For visual check of display edges when playing with different fonts
          if (id(media_source).state != "no source") {
              it.printf(64, 0, id(font_large), TextAlign::TOP_CENTER, "%.12s", id(media_source).state.c_str()); // 
          } else {
              it.printf(64, 0, id(font_large), TextAlign::TOP_CENTER, "Sonos"); // if no source list attribute, display Sonos Playlist instead
          }
          it.printf(0, 20, id(font_icons_medium), TextAlign::TOP_LEFT, "m"); // speaker on sign
          it.printf(21, 22, id(font_medium), TextAlign::TOP_LEFT, "%.f", id(sonos_volume).state);
          if (id(sonos_status).state != "playing") {
              it.printf(127, 18, id(font_icons_medium), TextAlign::TOP_RIGHT, "7"); // pause sign
          } else {
              it.printf(127, 18, id(font_icons_medium), TextAlign::TOP_RIGHT, "6"); // play sign
          }
          it.printf(107, 22, id(font_medium), TextAlign::TOP_RIGHT, "%s", id(sonos_status).state.c_str());
          it.printf(00, 53, id(font_icons_small), TextAlign::BOTTOM_LEFT, "s"); // star sign for master speaker
          it.printf(64, 53, id(font_small_bold), TextAlign::BOTTOM_CENTER, "%s", id(sonos_master).state.c_str());
          it.printf(00, 65, id(font_icons_small), TextAlign::BOTTOM_LEFT, "a"); // chain sign for slave speaker(s)
          it.printf(64, 65, id(font_small), TextAlign::BOTTOM_CENTER, "%s", id(sonos_slaves).state.c_str());
         
      - id: page2
        lambda: |-
          // it.rectangle(0, 0, 128, 64); For visual check of display edges when playing with different fonts
          if (id(media_source).state != "no source") {
              it.printf(64, 0, id(font_large), TextAlign::TOP_CENTER, "%.12s", id(media_source).state.c_str()); // 
          } else {
              it.printf(64, 0, id(font_large), TextAlign::TOP_CENTER, "Playlist"); // if no source list attribute, display 'Sonos Playlist' instead
          }
          it.printf(0, 20, id(font_icons_medium), TextAlign::TOP_LEFT, "m"); // speaker on sign
          it.printf(21, 22, id(font_medium), TextAlign::TOP_LEFT, "%.f", id(sonos_volume).state);
          if (id(sonos_status).state != "playing") {
              it.printf(127, 18, id(font_icons_medium), TextAlign::TOP_RIGHT, "7"); // pause sign
          } else {
              it.printf(127, 18, id(font_icons_medium), TextAlign::TOP_RIGHT, "6"); // play sign
          }
          it.printf(107, 22, id(font_medium), TextAlign::TOP_RIGHT, "%s", id(sonos_status).state.c_str());
          if (id(media_title).state != "None") {
              it.printf(73, 53, id(font_small), TextAlign::BOTTOM_CENTER, "%.24s", id(media_artist).state.c_str());
              it.printf(73, 65, id(font_small), TextAlign::BOTTOM_CENTER, "%.24s", id(media_title).state.c_str());
              it.printf(00, 53, id(font_icons_small), TextAlign::BOTTOM_LEFT, "A"); // person sign (artist)
              it.printf(01, 63, id(font_icons_14), TextAlign::BOTTOM_LEFT, "j"); // note sign (title)
          } else {
              it.strftime(64, 42, id(font_large), TextAlign::TOP_CENTER, "%H:%M:%S", id(esptime).now());
          }
      
      
      - id: page3
        lambda: |-
          // it.rectangle(0, 0, 128, 64); For visual check of display edges when playing with different fonts
          if (id(media_source).state != "no source") {
              it.printf(64, 0, id(font_large), TextAlign::TOP_CENTER, "%.12s", id(media_source).state.c_str()); // 
          } else {
              it.printf(64, 0, id(font_large), TextAlign::TOP_CENTER, "Sonos"); // if no source list attribute, display Sonos Playlist instead
          }
          it.printf(0, 20, id(font_icons_medium), TextAlign::TOP_LEFT, "m"); // speaker on sign
          it.printf(21, 22, id(font_medium), TextAlign::TOP_LEFT, "%.f", id(sonos_volume).state);
          if (id(sonos_status).state != "playing") {
              it.printf(127, 18, id(font_icons_medium), TextAlign::TOP_RIGHT, "7"); // pause sign
          } else {
              it.printf(127, 18, id(font_icons_medium), TextAlign::TOP_RIGHT, "6"); // play sign
          }
          it.printf(107, 22, id(font_medium), TextAlign::TOP_RIGHT, "%s", id(sonos_status).state.c_str());
          it.strftime(64, 42, id(font_large), TextAlign::TOP_CENTER, "%H:%M:%S", id(esptime).now());
       
      - id: page4   
        lambda: |-
          // it.rectangle(0, 0, 128, 64); For visual check of display edges when playing with different fonts
          if (id(media_source).state != "no source") {
              it.printf(64, 0, id(font_large), TextAlign::TOP_CENTER, "%.12s", id(media_source).state.c_str()); // 
          } else {
              it.printf(64, 0, id(font_large), TextAlign::TOP_CENTER, "Playlist"); // if no source list attribute, display Sonos Playlist instead
          }
          it.printf(0, 20, id(font_icons_medium), TextAlign::TOP_LEFT, "m"); // speaker on sign
          it.printf(21, 22, id(font_medium), TextAlign::TOP_LEFT, "%.f", id(sonos_volume).state);
          if (id(sonos_status).state != "playing") {
              it.printf(127, 18, id(font_icons_medium), TextAlign::TOP_RIGHT, "7"); // pause sign
          } else {
              it.printf(127, 18, id(font_icons_medium), TextAlign::TOP_RIGHT, "6"); // play sign
          }
          it.printf(107, 22, id(font_medium), TextAlign::TOP_RIGHT, "%s", id(sonos_status).state.c_str());
          it.printf(64, 42, id(font_large), TextAlign::TOP_CENTER, "Out: %.1f°C", id(outdoor_temp).state);
```

## Future improvement plans
- Improve/simplify templates
- Remove need for master entity everywhere in config files
- Display 'shuffle' and 'mute' sign
- Implement brightness control on display
- Implement display on/off (via local ESPHome PIR sensor)
- Add more information on display using a 2,9" E-ink display
- Add png file for current media playing for e-paper displays
- Use e-paper display as status display when Sonos is idle

Thank you Xavi for providing the perfect solution for some of my templating issues 👍😎

_[@htvekov](https://github.com/htvekov)_


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM0MDY2ODUxLC0xNjU3NDQ2OTkzLDIwNT
kyNzIzOTIsLTEyNjE5NDIyNzgsNDIyMTY3OTA5LDE0NzM1MDEw
NDcsMTQ3NDQ2NjI3NCwtNjQ5Nzk1MDcsLTUzNTk0MDQyOSwxNj
QzMDE4OTYzLDUwMTE1Njk1MiwxNjA5MjYzMTI3LC03MjgzMDYw
NjksMjU1NzE0NDA4LC0xMjk4Nzg0MDg2LDE3OTA0OTIwODcsLT
E2Nzk4NDkxMzUsLTE0MTM1NTgyNzYsLTIwNTg5NDQ1MDJdfQ==

-->