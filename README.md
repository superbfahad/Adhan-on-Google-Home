# Adhan-on-Google-Home
Adhan - Google Automation


sensor:
  - platform: rest
    name: 'Prayer Times'
    json_attributes:
      - data
    resource: 'http://api.aladhan.com/v1/timings?latitude=52.587904&longitude=-0.1458179&method=3'
    value_template: '{{ value_json["data"]["meta"]["method"]["name"].title() }}'
    scan_interval: 86400

  - platform: template
    sensors:
      fajr:
        friendly_name: 'Fajr Prayer Time'
        value_template: '{{ states.sensor.prayer_times.attributes.data.timings["Fajr"] | timestamp_custom("%H:%M") }}'
      dhuhr:
        friendly_name: 'Dhuhr Prayer Time'
        value_template: '{{ states.sensor.prayer_times.attributes.data.timings["Dhuhr"] | timestamp_custom("%H:%M") }}'
      asr:
        friendly_name: 'Asr Prayer Time'
        value_template: '{{ states.sensor.prayer_times.attributes.data.timings["Asr"] | timestamp_custom("%H:%M") }}'
      magrib:
        friendly_name: 'Magrib Prayer Time'
        value_template: '{{ states.sensor.prayer_times.attributes.data.timings["Maghrib"] | timestamp_custom("%H:%M") }}'
      isha:
        friendly_name: 'Isha Prayer Time'
        value_template: '{{ states.sensor.prayer_times.attributes.data.timings["Isha"] | timestamp_custom("%H:%M") }}'

automation:
  - alias: 'Fajr Athan'
    initial_state: true
    hide_entity: true
    trigger:
      - condition: template
        value_template: '{{ states.sensor.time.state == states("sensor.fajr") }}'
    action:
      - service: media_player.volume_set
        data_template:
          entity_id: media_player.living_room_speaker
          volume_level: 0.75
      - service: media_player.play_media
        data:
          entity_id: media_player.living_room_speaker
          media_content_id: https://s3.intahnet.co.uk/athan/fajr.mp3
          media_content_type: audio/mp3

  - alias: 'Athan'
    initial_state: true
    hide_entity: true
    trigger:
      - platform: template
        value_template: '{{ states.sensor.time.state == states("sensor.dhuhr") }}'
      - platform: template
        value_template: '{{ states.sensor.time.state == states("sensor.asr") }}'
      - platform: template
        value_template: '{{ states.sensor.time.state == states("sensor.maghrib") }}'
      - platform: template
        value_template: '{{ states.sensor.time.state == states("sensor.isha") }}'
    action:
      - service: media_player.volume_set
        data_template:
          entity_id: media_player.living_room_speaker
          volume_level: 0.75
      - service: media_player.play_media
        data:
          entity_id: media_player.living_room_speaker
          media_content_id: https://s3.intahnet.co.uk/athan/normal.mp3
          media_content_type: audio/mp3
