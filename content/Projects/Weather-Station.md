---
title : 'Weather Station'
date : 2024-08-18T09:34:25+01:00
draft : false
weather : true
preview: https://weather.lloyd.ws/static/img/webcam.png
cover:
    image: https://weather.lloyd.ws/static/img/webcam.png
    alt: Bishops Waltham Sky Now
    Caption: Bishops Waltham Sky Now
---
Data from the weather station in my garden. I have an old CCTV camera taking pictures of the sky every 5 minutes. I'm then autogenerating the webm of the previous days sky as timelapse. [My Weather Website](https://weather.lloyd.ws) the data is also posted to [Wundergound](https://www.wunderground.com/dashboard/pws/ISOUTH1316)
## Graphs
![Temperature](https://f001.backblazeb2.com/file/JLwebsite/temperature.png)
![Humidity](https://f001.backblazeb2.com/file/JLwebsite/Humidity.png)
![Pressure](https://f001.backblazeb2.com/file/JLwebsite/Pressure.png)
![Solar Radiance](https://f001.backblazeb2.com/file/JLwebsite/Solar-Radiance.png)

## Ecowitt GW2000
![ecowitt GW2001](/img/ecowitt.jpg)
The weather station is a [Ecowitt GW2000](https://shop.ecowitt.com/en-gb/products/wittboy).  Below is the python I'm using to grab the data. I'm then using Flask to export the data to Prometheus and grafana to render the graphs.


### Python Code
{{< highlight python "linenos=table" >}}
import requests

ECOWITT_HUB = 'x.x.x.x' # IP of ecowitt hub

def strip_unit(string):
    string = string.split(' ')
    output = string[0].replace('%','')
    return output

def get_weather_data():
    url = f'http://{ECOWITT_HUB}/get_livedata_info'
    headers = {
        'Accept': '*/*',
        'Accept-Language': 'en-GB,en;q=0.9',
        'Connection': 'keep-alive',
        'DNT': '1',
        'Referer': 'http://{ECOWITT_HUB}/liveData.html',
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36',
    }
    params = ''
    response = requests.get(url, params=params, headers=headers, verify=False)
    data = response.json()
    sensors =  {
                '3': 'Feels Like',
                '0x02': 'Outdoor Temperature',
                '0x07': 'Outdoor Humidity',
                '0x03': 'Dew point',
                '0x04': 'Wind chill',
                '0x05': 'Heat index',
                '0x0A': 'Wind Direction',
                '0x0B': 'Wind Speed',
                '0x0C': 'Gust Speed',
                '0x15': 'Solar Irradiance',
                '0x16': 'UV',
                '0x17': 'UV-Index',
                '0x18': 'Date and time',
                '0x19': 'Day Wind Max',
                '0x0D': 'Rain Event',
                '0x0E': 'Rain Rate',
                '0x0F': 'Rain hour',
                '0x10': 'Rain Day',
                '0x11': 'Rain Week',
                '0x12': 'Rain Month',
                '0x13': 'Rain Year',
                '0x14': 'Rain Totals',
    }
    
    units = {
                'Feels Like': '°C',
                'Outdoor Temperature': '°C',
                'Outdoor Humidity' : '%',
                'Dew point' : '°C',
                'Wind chill' : '°C',
                'Heat index' : '',
                'Wind Direction' : '° Degrees',
                'Wind Speed' : 'm/s',
                'Gust Speed' : 'm/s',
                'Solar Irradiance' : 'W/m²',
                'UV' : 'lx',
                'UV-Index' : '',
                'Date and time': '',
                'Day Wind Max': 'm/s',
                'Rain Event': 'mm',
                'Rain Rate': 'mm/hr',
                'Rain hour' : 'mm',
                'Rain Day': 'mm',
                'Rain Week': 'mm',
                'Rain Month': 'mm',
                'Rain Year': 'mm',
                'Rain Totals': 'mm',
                'Battery': '%'
        }
    output = {}
    output['Absolute Pressure']= { 'value' : strip_unit(data['wh25'][0]['abs']), 'unit': 'hPa' }
    output['Relative Pressure']= {'value' : strip_unit(data['wh25'][0]['rel']), 'unit': 'hPa' }
    for d in data['common_list']:
        output[sensors[d['id']]] = { "value" : strip_unit(d['val']), "unit": units[sensors[d['id']]] }
    for d in data['piezoRain']:
        output[sensors[d['id']]] = { "value" : strip_unit(d['val']), "unit": units[sensors[d['id']]] }
        if sensors[d['id']] == "Rain Year":
            battery = (float(d['battery'])/5)*100
            output['Battery'] = { "value" : str(battery), "unit": units['Battery'] }
    return output

{{< / highlight >}}